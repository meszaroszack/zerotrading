# CURRENT STATE

_Live pointer file. Overwrite at end of every session. Append-only history lives in `ai/summaries/DECISION-LOG.md`._

**Last updated:** 2026-05-20 01:15 ET
**Updated by:** Comet (browser agent) on behalf of meszaroszack

## CRITICAL: Know your market before doing ANYTHING

**`ai/guardrails/KALSHI-MARKET-REFERENCE.md` is REQUIRED READING.**

This repo targets TWO different Kalshi BTC markets. They have DIFFERENT structures:

| System | Market | Structure | Status |
|---|---|---|---|
| ZeroTrading Core | **KXBTCD** (hourly) | Ladder of strike prices; you pick a threshold; above/below that level | **LIVE** on Railway |
| ZeroTrading 15M | **KXBTC15M** (15-min) | Single strike = current price; simple up/down | Paper mode (planned) |

- **KXBTCD** = strike distance, threshold selection, price ladders. Live at `zerotrading-core-production.up.railway.app`.
- **KXBTC15M** = simple binary over/under. No ladders, no strike distance. See `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md`.

**Do NOT mix strategies between markets.** Do NOT apply 15M logic to KXBTCD or vice versa.

## Where we are

### ZeroTrading Core (KXBTCD - Hourly) - LIVE

- Deployed at `zerotrading-core-production.up.railway.app`.
- Connected to Kalshi exchange (LIVE, real money).
- Balance: ~$4.16 (as of last check).
- Status: CONNECTED, evaluating KXBTCD contracts, making NO_TRADE decisions (strike too close, net edge too low).
- Architecture in `docs/pdfs/` (execution FSM, accounting/fee-corrections).
- Execution FSM: IDLE -> ENTERING -> OPEN -> EXITING -> SETTLED.
- Uses BTC price from Binance for context (momentum, mean-reversion detection).

### ZeroTrading 15M (KXBTC15M) - PLANNED

- AI-first repo scaffold complete: `ai/`, `docs/`, `ops/runbooks/`, `research/`, `.cursor/rules/`.
- MASTER-PROMPT v2 active at `ai/prompts/MASTER-PROMPT.md`.
- **Deep-dived two prior 15m builds:**
  - `research/adapted/trader-retro-15m.md` - TS swing bot, monolithic, InMemory, signals on BTC spot.
  - `research/adapted/kalshi-15m-bot-15m.md` - Python multi-strategy, modular, WS feed, RiskGuard, OrderManager.
- **Verdict:** kalshi-15m-bot is the primary donor codebase. Both share fatal flaw: no persistent state, no exchange reconciliation.
- **Mystic Bot analysis complete:**
  - `research/mystic/mystic-performance-summary.md` v2 - Directional bias detection edge analysis.
  - Mystic live data (April 12-15, 2026): 134W/3L, ~97.8% win rate on real Kalshi fills.
  - Assessment: Credible proof-of-concept using directional momentum on over/under contracts.
- **Strategy v3 deployed:**
  - `docs/STRATEGY-KXBTC15M.md` v3 with correct over/under model.
- **Performance governance:**
  - `docs/PERFORMANCE-RECORDS.md` with PERF-001 (Mystic data), verification policy.
- No code in zerotrading repo yet for 15M. No live deploy.

### Guardrails

- `ai/guardrails/KALSHI-MARKET-REFERENCE.md` - Master market taxonomy (KXBTCD vs KXBTC15M vs other markets).
- `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md` - 15M-specific guardrails (over/under, not range).

## Key findings

### From prior builds (both markets)
1. Both bots use InMemoryStorage - restart = orphaned Kalshi positions.
2. P&L computed locally from bid prices, not actual fill prices. Fees not accounted.
3. Three critical fixes for any port: persistent state, fill verification + fee accounting, exchange reconciliation.

### From Mystic analysis (KXBTC15M only)
1. KXBTC15M is over/under, not range. Edge is directional, not range containment.
2. ~98% win rate suggests selective momentum trading within windows.
3. Fee drag significant on cheap contracts.
4. Minimum 500+ trades needed before edge claims.

### From live KXBTCD bot
1. Bot is connected and evaluating but mostly deciding NO_TRADE (edge not sufficient).
2. Uses strike distance and implied probability vs estimated probability.
3. Balance ~$4.16 with 0 completed cycles so far.

## Decisions pending (user input needed)

1. **Language for 15M:** Python (recommended) vs TypeScript.
2. **Starting strategy parameters:** Use STRATEGY-KXBTC15M.md v3 defaults, or customize?
3. **More builds to review?** User mentioned there are more.
4. **Port approach:** Copy kalshi-15m-bot src/ structure and add Supabase, or clean rewrite?
5. **KXBTCD tuning:** Should the live bot's edge thresholds be adjusted to take more trades?

## Next concrete step

1. User confirms language + starting strategy + whether more builds to review.
2. Port kalshi-15m-bot modular structure into `src/` in this repo.
3. Add Supabase persistence layer.
4. Add boot reconciliation.
5. Implement directional momentum strategy per STRATEGY-KXBTC15M.md v3.
6. Start in paper mode.

## Do not regress

- **KXBTCD and KXBTC15M are different markets.** See `ai/guardrails/KALSHI-MARKET-REFERENCE.md`.
- **KXBTC15M is over/under, not range.** See `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md`.
- Execution FSM (see `docs/pdfs/zerotrading-core-execution-statemachine.pdf`).
- Accounting / fee corrections (see `docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf`).
- OPSEC: no secrets, no account IDs in repo.
- Mode discipline: paper -> safe-live -> live.
- Performance verification policy: no go-live without exchange-reconciled, fee-adjusted PnL.
