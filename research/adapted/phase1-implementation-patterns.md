# Phase 1 Implementation Patterns
## ZeroTrading x15minBTC — Design Decisions & Lessons from the Skeleton Build

**Session:** 2026-05-21 21:00 ET  
**Author:** Perplexity Computer (Comet) on behalf of meszaroszack  
**Beta commit:** `d205ec4` (main branch, zerotradingx15minbtc)  
**Status:** Implemented and syntax-verified; not yet live

---

## Why This Document Exists

Every prior build's fatal flaws came from undocumented or inconsistent design decisions that agents and developers couldn't reliably inherit. This document captures the Phase 1 skeleton's explicit decisions — the kind that would otherwise live only in chat — so future sessions and agents inherit them without re-litigating.

---

## Decision 1: Zero-Drift Black-Scholes for Terminal Probability

**What:** Use `P(BTC ≥ K) = N(d2)` with zero drift.

```
d2 = (ln(S/K) − 0.5 · σ² · T) / (σ · √T)
```

**Why zero drift:** BTC 15-minute drift is dominated by diffusion noise. Estimating drift from 1m candles introduces more error than it removes. The zero-drift assumption is consistent with risk-neutral pricing and produces well-calibrated probabilities against historical KXBTC15M settlements.

**Alternatives rejected:**
- Historical drift from 20-candle window: too noisy, would require much longer window
- Implied risk-neutral drift from term structure: Kalshi doesn't publish term structure for 15m markets
- ML model: overkill for Phase 1; adds parameter estimation risk without proven edge

**Where implemented:** `src/strategy/kxbtc15m.py → terminal_probability_yes()`

**Risk:** If BTC is in a strong trending regime, zero drift underestimates directional momentum. The vol regime gate (CALM/MODERATE only) partially mitigates this — trending regimes tend to produce ELEVATED/EXTREME vol.

---

## Decision 2: Taker-Only Fee Model in Paper Mode

**What:** All paper fills modeled as taker (worst-case fee).

**Why:** Paper mode should produce conservative P&L estimates. If the model is profitable with taker fees, live mode (which may get some maker fills at lower cost) will be at least as good. Maker fill probability is hard to estimate without live market data.

**Specific formula:**
```
fee = ceil(0.07 * contracts * P * (1 - P))
```

**Where implemented:** `src/accounting/fees.py → kalshi_fee()`

**Live mode plan:** Use `taker_fill_cost_dollars` from the WS `fill` event, which includes actual fees and supersedes this estimate.

---

## Decision 3: Supabase-Backed FSM, No In-Memory State

**What:** Every FSM state transition writes to Supabase BEFORE the function returns. The in-memory `ExecutionFSM` object holds only `active_position_id` — a pointer, not state.

**Why:** Every prior build died because Railway restarts cleared in-memory state, orphaning positions. With DB-backed FSM:
- Restart → `fsm.boot()` reads non-IDLE rows → state restored
- No special restart handling code
- Reconciliation catches any discrepancy independently

**Transition guarantee:** If a DB write fails, the transition raises and the caller handles it. The system never advances state without confirmed DB write. No two-phase commits needed for a single-table write.

**Idempotency guard:** Before `EVALUATING` insert, the FSM checks for an existing record with `(market_ticker, window_open_time)` in a non-IDLE state. If found, it raises `FSMTransitionError`. This is the primary duplicate-trade guard.

**Where implemented:** `src/execution/fsm.py → ExecutionFSM`

---

## Decision 4: `client_order_id` as Primary Idempotency Key

**What:** Every order (real or simulated) gets a UUID assigned as `client_order_id` before submission.

**Why:** If the network drops after submitting but before receiving the response, a retry without `client_order_id` would create a duplicate order. Kalshi's API is idempotent on `client_order_id` — submitting the same ID twice returns the existing order, not a new one.

**Scope:** Paper mode generates and stores `client_order_id` in `positions.client_order_id` even though no real order is placed. This ensures the field is populated for the live mode migration path.

**Where implemented:** `src/execution/fsm.py → transition_to_entering()`, `src/execution/paper.py → simulate_entry()`

---

## Decision 5: Consecutive-Loss Cooldown in Memory (Not Persisted)

**What:** After 3 consecutive losses, a 1-hour cooldown is enforced. The counter lives in `PaperTrader._consecutive_losses` — resets on restart.

**Why it's intentional:** A deliberate restart can override a cooldown. This is a feature, not a bug — if you restart the bot during cooldown to change parameters, you don't want the cooldown to follow you. The cooldown is a runtime safety valve, not a persistent audit constraint.

**Risk:** An unplanned crash during cooldown would bypass it. Acceptable for Phase 1; Phase 2 can persist to Supabase if needed.

**Where implemented:** `src/execution/paper.py → PaperTrader.settle()`

---

## Decision 6: `KALSHI_MARKET_TICKER` from Env Var (Phase 1)

**What:** The active KXBTC15M contract ticker is manually set via `KALSHI_MARKET_TICKER` environment variable.

**Why:** KXBTC15M tickers are short-lived (15 minutes per window). The ticker format is `KXBTC-YYMMDDHHM-T{strike}`. For Phase 1 paper testing, the user rotates this manually. Auto-discovery from the REST catalog is a Phase 2 feature.

**Strike parsing:** `main.py → _parse_strike_from_ticker()` extracts the strike from the ticker suffix (`-T100000` → 100000.0).

---

## Decision 7: Async Class-Based REST Client

**What:** `KalshiRESTClient` is an async class using `httpx.AsyncClient` per-call.

**Why class-based (not module functions):** The original skeleton used module-level functions. When `main.py` needed to pass the REST client to reconciliation, strategy, and the loop, module functions required no injection — which is fine for a single process but makes testing and mocking impossible. Class-based lets you pass `rest: KalshiRESTClient` explicitly.

**Why per-call AsyncClient (not persistent session):** Kalshi REST calls are low frequency (reconciliation every 60s, settlement polls). Persistent sessions add complexity and Kalshi's rate limiter is per-IP, not per-connection. Per-call is fine at this volume.

**Where implemented:** `src/ingestion/kalshi_rest.py → KalshiRESTClient`

---

## Decision 8: 30-Second Warm-Up Delay at Boot

**What:** After starting WS tasks, `main.py` sleeps 30 seconds before entering the evaluation loop.

**Why:** `StrategyContext.is_ready()` requires 20 closed 1m candles. At 1 candle/minute, that's ~20 seconds minimum. The 30-second wait provides buffer and avoids a tight loop of `INSUFFICIENT_DATA` skip decisions that would fill logs with noise.

**`is_ready()` still gates independently:** The warm-up is a UX improvement, not a correctness dependency. The strategy still checks `context.is_ready()` as Gate 1 on every evaluation.

---

## Decision 9: `shared.btc_spot` as Mutable List

**What:** BTC spot price is stored in a single-element list `shared.btc_spot = [0.0]` shared across the Binance callback, the evaluation loop, and the health logger.

**Why a list:** Python asyncio coroutines run on the same thread, so true thread safety isn't needed. But floats are immutable — reassigning `btc_spot = new_value` in one coroutine wouldn't be visible in another that holds a reference to the old value. A mutable list (`btc_spot[0] = new_value`) updates in-place and is immediately visible everywhere.

**Alternative:** `asyncio.Queue` is cleaner but adds overhead and buffering complexity for a value we only ever want the latest of.

---

## Decision 10: Feed Health Gates Every Evaluation Cycle

**What:** `feed_health.is_healthy()` is checked before every evaluation in the main loop. Stale feeds (Kalshi > 30s, Binance > 90s) block evaluation.

**Why 90s for Binance:** 1m candles arrive every ~60 seconds. 90s allows one missed message before triggering. The Binance WS client also has its own 30-second watchdog that will reconnect independently — the feed health check is a second layer.

**Why this matters:** If Kalshi WS is stale, the orderbook snapshot is outdated. If Binance WS is stale, sigma is stale. Trading on stale data is worse than not trading.

---

## Patterns Inherited from Prior Builds

| Pattern | Source build | How it's used here |
|---------|-------------|-------------------|
| Black-Scholes terminal probability | kalshi-btcd-trader | `kxbtc15m.py`, zero-drift adaptation |
| Supabase persistence | alpha-bot | All state tables; daily_pnl; decisions archive |
| `client_order_id` idempotency | kalshi-btcd-trader, probablyprofit | Every order, paper and live |
| Boot reconciliation | PERSISTENCE-RECONCILIATION.md | `reconcile.py → boot_reconcile()` |
| WS + REST dual reconciliation | probablyprofit, PERSISTENCE-RECONCILIATION.md Rule 5 | 60s periodic REST recon in loop |
| Vol regime classification | kalshi-btcd-trader | `context.py → VolRegime` |
| `taker_fill_cost_dollars` for P&L | kalshi-public-docs (WS fill event) | `paper.py`; live mode will use directly |
| Exponential backoff WS reconnect | kalshi-trader, Mystic research | `kalshi_ws.py`, `binance_ws.py` |
| Fee formula `ceil(0.07*C*P*(1-P))` | KALSHI-MARKET-REFERENCE.md | `fees.py → kalshi_fee()` |

---

## Patterns Explicitly Rejected

| Pattern | Why rejected |
|---------|-------------|
| Technical indicators (RSI, MACD, BB) | Alpha-bot failure analysis: dubious for 15-min binary outcome prediction |
| In-memory state / file persistence | Every prior build failure root cause |
| Fill price from orderbook snapshot | Use actual `taker_fill_cost_dollars` from fill event |
| Multiple concurrent positions | `MAX_CONCURRENT_POSITIONS=1`; binary concentration risk |
| Drift term in BS model | Too noisy from 20-candle window; zero-drift better calibrated |
| Maker-only strategy | Complex; Phase 1 uses taker; Phase 2 revisit |

---

## Known Phase 2 Work Items (documented here to prevent loss)

1. **Live order execution** — `src/execution/orders.py` is a stub; implement `LiveOrderManager.place_order()` using `KalshiRESTClient.place_limit_order()` + WS fill confirmation
2. **Macro gate** — Integrate FOMC/CPI/NFP calendar; block ±30min of high-impact events
3. **Auto-ticker discovery** — Query Kalshi REST `/markets?ticker=KXBTC15M&status=open` to find the active market; remove `KALSHI_MARKET_TICKER` manual env var requirement
4. **Consecutive-loss cooldown persistence** — Write to `daily_pnl` or a dedicated `guardrail_state` table so it survives restarts
5. **Maker order logic** — If spread is wide enough, try to rest a limit order at mid; use `maker_fill_cost_dollars` from WS for P&L
6. **Backtesting harness** — Replay historical `market_snapshots` and `decisions` tables against the strategy module offline
7. **Hour-bias layer** — From `kalshi-trader` research: certain hours have predictable directional bias; implement as confidence multiplier
