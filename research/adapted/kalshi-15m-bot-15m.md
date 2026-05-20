# kalshi-15m-bot - KXBTC15M Multi-Strategy Bot (Python)

**Source URL:** https://github.com/meszaroszack/kalshi-15m-bot (private)
**Live:** https://kalshi-15m-bot-production.up.railway.app/
**Language:** Python (asyncio + Flask dashboard)
**Last commit:** Claude Agent, 2 weeks ago (dynamic momentum exits)
**Pulled by:** Comet on 2026-05-19

## What it does
Full-featured async trading bot with WebSocket market feed, modular strategy system (6 strategies), dedicated OrderManager with position tracking, RiskGuard pre-trade checks, and a live Flask dashboard with pause/resume and credential hot-swap. Significantly more mature than trader-retro.

## Architecture (properly modular)
- `bot.py` - TradingBot orchestrator: market refresh loop (30s), trading loop (configurable poll), WS feed, dashboard integration
- `src/client/rest.py` - Kalshi REST client (typed, RSA auth)
- `src/client/auth.py` - key loading from string or file
- `src/feed/ws_feed.py` - Kalshi WebSocket v2 feed with MarketState objects (orderbook, last_price, spread, close_time)
- `src/orders/manager.py` - 587-line OrderManager: entry, multiple exit types, position dataclass, settlement handling, fill verification, PnL tracking
- `src/risk/guard.py` - RiskGuard: spread check (reject if >10), expiry guard, duplicate ticker guard, momentum re-entry cooldown
- `src/strategies/` - 6 strategies:
  - `last90` - entry in last 90s of market
  - `threshold` - fixed-cents entry/target/stop
  - `mean_revert` - fade extremes
  - `momentum` - dynamic exits with hard%, arm-trail, fade detection, settlement-hold
  - `manual_override` - manual entries via dashboard
  - `scalp` - quick in/out with tight stops
- `config/settings.py` - BotConfig dataclass: all parameters, multi-series support (BTC, ETH, SOL)
- `dashboard.py` - Flask web UI: live status, trade log, signal log, pause/resume, auth input
- `backtest.py` - backtesting harness
- `tests/` - test suite

## What is reusable for ZeroTrading
- **Entire modular architecture:** client/feed/orders/risk/strategies separation is exactly what ZeroTrading needs. This is the closest thing to a working production bot.
- **OrderManager pattern:** Position dataclass with entry/exit tracking, fill verification via order polling, multi-exit-type support (take-profit, stop-loss, momentum trailing, settlement-hold). Port this as the core of ZeroTrading's execution layer.
- **RiskGuard pattern:** Pre-trade gating with spread, expiry, and position-duplicate checks. Maps directly to `.cursor/rules/trading-safety.mdc`.
- **WebSocket feed + MarketState:** Real orderbook data, not just BTC spot. This is the right signal source.
- **Multi-strategy plugin system:** STRATEGY_MAP pattern with a base Signal class. Clean way to swap strategies without touching the engine.
- **Settlement-hold logic:** Suspends trailing exits in final seconds if winning. Smart for binary markets where holding through expiry = max payout.
- **Config as dataclass with env-var fallbacks:** Clean, Railway-friendly.
- **Dashboard with pause/resume:** Operational necessity for any live bot.
- **Backtest harness:** Even if basic, having one at all is ahead of trader-retro.

## What is wrong / not portable
- **No persistent storage:** Like trader-retro, positions are in-memory. Restart = orphaned positions on Kalshi.
- **Accounting still local:** `daily_pnl_cents` computed from local fill assumptions, not reconciled against exchange. Same drift risk as trader-retro.
- **Claude-authored commits with drift:** Last commits by "Claude Agent" add increasingly complex exit logic (hard%/arm-trail/fade). Each feature added more code paths but possibly introduced edge cases. Settlement-hold guard was a bug fix 2 weeks ago - suggests the dynamic exits were prematurely closing winning positions.
- **No reconciliation on boot:** If the process restarts, open Kalshi positions are invisible to the bot.
- **P&L = (exit_price - entry_price) * count:** Doesn't account for Kalshi fees (typically 1c/contract or percentage). Over many trades this compounds into the accounting discrepancy vs the actual Kalshi account balance.
- **Market orders for exits:** `_close` uses market orders to ensure execution, but market orders on illiquid Kalshi markets can have severe slippage.
- **BotConfig defaults not tuned:** Config exists but the user noted execution was sporadic - likely strategy parameters (entry thresholds, stop distances) weren't calibrated to actual KXBTC15M behavior.

## Port plan
- **Keep (high value):** Entire src/ module structure. OrderManager position lifecycle. RiskGuard gating. WebSocket MarketState feed. Multi-strategy plugin. Settlement-hold concept. Dashboard pause/resume. Backtest harness skeleton.
- **Rewrite:** Add Supabase persistence layer under OrderManager. Add boot reconciliation (fetch open positions from Kalshi, rebuild local state). Add fee accounting (query fills for actual fees paid). Replace market-order exits with limit+slip or IOC orders.
- **Enhance:** Reconciliation loop (compare local PnL vs Kalshi account balance every N minutes). Add the execution FSM from ZeroTrading core PDFs as the state machine wrapping OrderManager. Connect 24/7 monitoring layer as the data source for MarketState.
- **Drop:** In-memory-only state. Claude-drift exit complexity (simplify to: hard stop + time stop + settlement hold, then re-add trailing only after paper-mode validation).
- DECISION-LOG entry: pending

## Open questions
- Which strategy was running in production? User didn't specify. Need to check Railway env vars or dashboard state.
- What were the actual win/loss numbers from the Kalshi account? Bot dashboard may show one thing, Kalshi another.
- The momentum strategy's settlement-hold is interesting but complex. Should ZeroTrading start with the simpler threshold or last90 strategy and add momentum later?
- Python vs TS for ZeroTrading: this bot is Python, trader-retro is TS. ZeroTrading repo currently has no code. Decision needed.

## Comparison: trader-retro vs kalshi-15m-bot
| Dimension | trader-retro | kalshi-15m-bot |
|---|---|---|
| Language | TypeScript | Python |
| Architecture | Monolithic single file | Properly modular (client/feed/orders/risk/strategies) |
| Signal source | BTC spot price (Coinbase) | Kalshi orderbook via WebSocket |
| Strategies | 3 (momentum, mean-revert, swing) | 6 (threshold, mean-revert, momentum, last90, manual, scalp) |
| Risk management | None (no loss cap) | RiskGuard (spread, expiry, duplicate, daily loss) |
| Persistence | In-memory only | In-memory only |
| Fill verification | None | Polls order status |
| Dashboard | React/SSE retro theme | Flask with pause/resume + auth |
| Maturity | Prototype | Near-production |
| Biggest weakness | Signal source + no persistence | No persistence + accounting drift |

**Verdict:** kalshi-15m-bot is the primary donor for ZeroTrading KXBTC15M. trader-retro contributes the Kalshi API client pattern and the swing-detection idea. Both share the same fatal flaw: no persistent state and no exchange reconciliation.
