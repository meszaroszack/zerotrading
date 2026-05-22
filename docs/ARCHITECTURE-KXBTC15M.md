# ZeroTrading x15minBTC — Implementation Architecture

> **Status:** Phase 1 skeleton complete (2026-05-21)  
> **Beta repo:** [meszaroszack/zerotradingx15minbtc](https://github.com/meszaroszack/zerotradingx15minbtc)  
> **Source of truth for implementation:** beta repo `docs/ARCHITECTURE.md`, `src/`  
> **This document:** canonical reference in the parent repo — updated from beta after sessions

---

## What This System Does

Trades the **KXBTC15M** Kalshi market: a simple binary over/under on BTC price at the close of each 15-minute window. The single strike equals BTC spot at window open. YES pays $1.00 if BTC finishes at or above the strike; NO pays $1.00 if below.

The system:
1. Streams live BTC price from Binance (1m klines)
2. Computes the "true" terminal probability using Black-Scholes
3. Compares to market-implied probability (Kalshi orderbook mid)
4. Enters when edge exceeds a threshold (default 8 cents)
5. Holds to settlement
6. Books P&L only after confirmed settlement

---

## 9-Layer Architecture

```
┌─────────────────────────────────────────────────────────┐
│  Layer 1: Market Ingestion                              │
│    kalshi_rest.py   — REST: balance, positions, orders  │
│    kalshi_ws.py     — WS: orderbook_delta, user_orders, fill │
│    binance_ws.py    — WS: BTC 1m klines                 │
│    feed_health.py   — Dual-feed liveness monitor        │
├─────────────────────────────────────────────────────────┤
│  Layer 2: Strategy Context                              │
│    context.py       — Rolling 20-candle sigma           │
│                       Vol regime: CALM/MODERATE/ELEVATED/EXTREME │
│                       Macro gate (Phase 1: stub)        │
├─────────────────────────────────────────────────────────┤
│  Layer 3: KXBTC15M Strategy Module                      │
│    kxbtc15m.py      — Terminal probability (BS), 7 gates│
│    sizing.py        — Contract sizing (balance/risk/floor) │
├─────────────────────────────────────────────────────────┤
│  Layer 4: Risk + Sizing                                 │
│    sizing.py        — MAX_CONTRACTS, MAX_RISK, FLOOR    │
├─────────────────────────────────────────────────────────┤
│  ╔═══════════════════════════════════════════════════╗  │
│  ║  Layer 5: Execution FSM  ← PROTECTED CORE        ║  │
│  ║    fsm.py    — Supabase-backed state machine      ║  │
│  ║    paper.py  — Simulated fills, real records      ║  │
│  ║    orders.py — Live order stub (Phase 2)          ║  │
│  ╚═══════════════════════════════════════════════════╝  │
├─────────────────────────────────────────────────────────┤
│  ╔═══════════════════════════════════════════════════╗  │
│  ║  Layer 6: Reconciliation  ← PROTECTED CORE       ║  │
│  ║    reconcile.py — Boot + periodic recon           ║  │
│  ╚═══════════════════════════════════════════════════╝  │
├─────────────────────────────────────────────────────────┤
│  ╔═══════════════════════════════════════════════════╗  │
│  ║  Layer 7: Accounting + Fee Handling  ← PROTECTED ║  │
│  ║    fees.py  — ceil(0.07 * C * P * (1-P))         ║  │
│  ║    pnl.py   — Realized P&L, daily summary         ║  │
│  ╚═══════════════════════════════════════════════════╝  │
├─────────────────────────────────────────────────────────┤
│  Layer 8: Operator Logs / Runtime Display               │
│    logging.py   — Structured JSON to stdout (Railway)   │
│    health.py    — Background asyncio health ticker      │
├─────────────────────────────────────────────────────────┤
│  Layer 9: Archive / Analytics                           │
│    Supabase: decisions, market_snapshots, daily_pnl     │
└─────────────────────────────────────────────────────────┘
```

---

## File Map

```
zerotradingx15minbtc/
├── main.py                          ← Entry point
├── requirements.txt                 ← websockets, httpx, supabase, cryptography
├── Procfile                         ← worker: python main.py
├── railway.toml                     ← Railway deploy config
├── .env.example                     ← All required env vars documented
├── src/
│   ├── db/
│   │   ├── supabase.py              ← Singleton client (env: SUPABASE_URL/ANON_KEY)
│   │   └── schema.sql               ← Full DDL — 6 tables (apply once to Supabase)
│   ├── ingestion/
│   │   ├── kalshi_rest.py           ← Async class: KalshiRESTClient
│   │   ├── kalshi_ws.py             ← KalshiWebSocketClient (3 channels)
│   │   ├── binance_ws.py            ← BinanceWebSocketClient (1m klines)
│   │   └── feed_health.py           ← FeedHealthMonitor
│   ├── strategy/
│   │   ├── context.py               ← StrategyContext (sigma, regime)
│   │   ├── kxbtc15m.py              ← evaluate() → StrategyDecision
│   │   └── sizing.py                ← compute_contracts()
│   ├── execution/
│   │   ├── fsm.py                   ← ExecutionFSM (5 states)
│   │   ├── paper.py                 ← PaperTrader (full simulation)
│   │   └── orders.py                ← LiveOrderManager (Phase 2 stub)
│   ├── accounting/
│   │   ├── reconcile.py             ← boot_reconcile(), periodic_reconcile()
│   │   ├── fees.py                  ← kalshi_fee(), entry_cost_usd()
│   │   └── pnl.py                   ← book_realized_pnl(), get_daily_pnl()
│   └── ops/
│       ├── logging.py               ← Logger (structured JSON)
│       └── health.py                ← health_loop() asyncio task
```

---

## Supabase Schema (6 Tables)

| Table | Layer | Purpose |
|-------|-------|---------|
| `positions` | 5 (FSM) | One row per trade window; FSM state, fill details |
| `trades` | 7 (Accounting) | Completed trades; P&L, one row per position (UNIQUE) |
| `decisions` | 9 (Archive) | Every TRADE and NO_TRADE evaluation; skip reasons |
| `market_snapshots` | 9 (Archive) | Orderbook state at decision time |
| `reconciliation_events` | 6 (Recon) | Audit log of every boot + periodic check |
| `daily_pnl` | 9 (Archive) | Daily P&L summary; upserted after each trade |

---

## FSM State Machine

```
  IDLE ──────────────────────────────────────────┐
    │ new 15m window, edge found                  │
    ▼                                             │
  EVALUATING ──(no edge / skip gate)──────────────┤ (abort_to_idle)
    │ order placed (paper: simulated)             │
    ▼                                             │
  ENTERING ──(order timeout / cancel)─────────────┤
    │ fill confirmed                              │
    ▼                                             │
  POSITIONED                                      │
    │ T-60s before window close                  │
    ▼                                             │
  SETTLING                                        │
    │ settlement result received                  │
    └─────────────────────────────────────────────┘
```

**Key guarantees:**
- Every state transition writes to Supabase BEFORE returning
- Duplicate-trade guard: checks `(market_ticker, window_open_time)` before EVALUATING insert
- P&L only booked at `SETTLING → IDLE` (never before confirmed settlement)
- `abort_to_idle()` available from any state for emergency recovery

---

## Boot Sequence

```
1. validate_env()           → fail-fast if any required var missing
2. fsm.boot()               → read non-IDLE positions from Supabase; restore state
3. boot_reconcile(rest)     → compare Supabase positions vs Kalshi live positions
     CLEAN    → proceed
     DIVERGED → update Supabase qty from Kalshi truth; proceed
     GHOST    → mark position SETTLING; proceed
     ORPHANED → raise ReconcileError; HALT (manual review required)
4. Start WS tasks           → kalshi_ws, binance_ws, health_loop (background)
5. Wait 30s warmup          → let Binance deliver ~30 candles for sigma computation
6. Enter paper loop         → evaluate every 5s
```

---

## Probability Model

```
P(BTC_terminal ≥ strike | S, σ, T) = N(d2)

d2 = (ln(S/K) − 0.5 · σ² · T) / (σ · √T)

Where:
  S  = BTC spot (Binance last closed candle)
  K  = strike (BTC spot at window open)
  σ  = per-minute realized volatility (std of log returns, rolling 20 candles)
  T  = minutes remaining in window

Drift = 0 (zero-drift approximation; negligible over 15 minutes)
```

**Implied probability:** mid-price of YES orderbook = `(yes_bid + yes_ask) / 2`

**Edge:** `P_model − P_implied`  
**Trade direction:** positive edge → buy YES; negative edge → buy NO  
**Threshold:** `|edge| ≥ EDGE_THRESHOLD` (default 0.08)

---

## 7 Skip Gates (in order)

| # | Gate | Fail condition | Reason code |
|---|------|---------------|-------------|
| 1 | Sufficient data | `context.candles_loaded < 20` | `INSUFFICIENT_DATA` |
| 2 | Vol regime EXTREME | annualized σ ≥ 150% | `EXTREME_VOL` |
| 3 | Vol regime ELEVATED | 80% ≤ annualized σ < 150% | `ELEVATED_VOL` |
| 4 | Macro gate | calendar event blocking (Phase 1: never fails) | `MACRO_BLOCKED` |
| 5 | Entry window | `minutes_into_window > 5` | `ENTRY_WINDOW_CLOSED` |
| 6 | Spread | `yes_ask − yes_bid > $0.04` | `SPREAD_TOO_WIDE` |
| 7 | Edge threshold | `|P_model − P_implied| < 0.08` | `EDGE_INSUFFICIENT` |

Every NO_TRADE decision is written to the `decisions` Supabase table with the exact fail reason.

---

## Fee Formula

```python
fee = ceil(0.07 * contracts * P * (1 - P))
```

Where `P` = YES price as decimal. Applied to taker orders.  
Maker orders receive a rebate; Phase 1 models all fills as taker (conservative).

Source: Kalshi fee schedule + `ai/guardrails/KALSHI-MARKET-REFERENCE.md`

---

## Volatility Regime Classification

| Regime | Annualized σ | Trade allowed |
|--------|-------------|---------------|
| CALM | < 40% | ✅ Yes |
| MODERATE | 40% – 80% | ✅ Yes |
| ELEVATED | 80% – 150% | ❌ No |
| EXTREME | ≥ 150% | ❌ No |

Default when insufficient data: EXTREME (fail conservative).

---

## Paper Mode vs Live Mode

| Dimension | Paper | Live (Phase 2) |
|-----------|-------|----------------|
| Order placement | Simulated at ask price | Real Kalshi REST order |
| Fill confirmation | Immediate (no WS event) | WS `fill` channel event |
| Supabase records | Full, identical to live | Full, identical to paper |
| P&L | Computed from sim fill | Computed from `taker_fill_cost_dollars` |
| Settlement | Poll Kalshi REST market result | Same |
| Risk guardrails | Active | Active |

Paper mode is NOT a stub. All records are real. The only difference is no real money moves.

---

## Guardrails (Paper Mode)

| Guardrail | Default | Env var |
|-----------|---------|---------|
| Daily loss cap | -$50 | `DAILY_LOSS_CAP_USD` |
| Safety floor | $50 balance minimum | `SAFETY_FLOOR_USD` |
| Consecutive losses | 3 losses → 1hr cooldown | (hardcoded, Phase 1) |
| Max contracts / trade | 10 | `MAX_CONTRACTS_PER_TRADE` |
| Max risk / trade | $20 | `MAX_RISK_PER_TRADE_USD` |

---

## Required Environment Variables

```
# Kalshi
KALSHI_API_KEY_ID
KALSHI_API_PRIVATE_KEY       # RSA PEM, \n escaped for Railway
KALSHI_ENV                   # demo | production
KALSHI_MARKET_TICKER         # e.g. KXBTC-25MAY2112-T100000

# Supabase
SUPABASE_URL
SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY

# App
APP_MODE                     # paper | safe-live | production

# Optional (defaults shown)
BINANCE_WS_URL               # wss://stream.binance.com:9443/ws/btcusdt@kline_1m
LOG_LEVEL                    # INFO
DAILY_LOSS_CAP_USD           # 50
SAFETY_FLOOR_USD             # 50
EDGE_THRESHOLD               # 0.08
MAX_CONTRACTS_PER_TRADE      # 10
MAX_RISK_PER_TRADE_USD       # 20
```

---

## What Is NOT Done (Phase 2 scope)

- Live order execution (`orders.py` is a stub that raises `NotImplementedError`)
- Macro gate (FOMC/CPI/NFP calendar integration)
- Auto-discovery of active KXBTC15M market ticker from Kalshi REST catalog
- Backtesting harness
- Consecutive-loss cooldown persistence (currently in-memory; resets on restart)
- Maker order logic (Phase 1 always models taker)

---

## Cross-References

| Document | Location |
|----------|---------|
| Strategy spec (v3) | `docs/STRATEGY-KXBTC15M.md` |
| Market structure guardrail | `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md` |
| Persistence/recon rules | `ai/guardrails/PERSISTENCE-RECONCILIATION.md` |
| Kalshi API reference | `ai/guardrails/KALSHI-MARKET-REFERENCE.md` |
| Beta repo | [meszaroszack/zerotradingx15minbtc](https://github.com/meszaroszack/zerotradingx15minbtc) |
| Beta DECISION-LOG | beta `ai/summaries/DECISION-LOG.md` |
| Beta session summary (Phase 1) | beta `ai/summaries/2026-05-21-2100-phase1-skeleton.md` |
