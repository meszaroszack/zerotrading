# CURRENT STATE

_Live pointer file. Overwrite at end of every session. Append-only history lives in `ai/summaries/DECISION-LOG.md`._

**Last updated:** 2026-05-20 01:00 ET
**Updated by:** Comet (browser agent) on behalf of meszaroszack

## CRITICAL: Read before doing ANY KXBTC15M work

**`ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md` is REQUIRED READING.**

KXBTC15M is a **simple binary over/under contract.** One strike (= BTC spot at window open). Two outcomes: above or below. That's it. NO ranges, wedges, bands, convergence zones, or strike distances. AI agents consistently get this wrong - see guardrail doc for why and how to prevent it.

## Where we are

- AI-first repo scaffold complete: `ai/`, `docs/`, `ops/runbooks/`, `research/`, `.cursor/rules/`.
- Authoritative PDFs in `docs/pdfs/` (architecture, execution FSM, accounting/fee-corrections).
- MASTER-PROMPT v2 active at `ai/prompts/MASTER-PROMPT.md`.
- Research scaffolds live: `research/notes/`, `research/adapted/`, `research/mystic/`.
- `docs/SYSTEM-SYNTHESIS.md` seeded with initial cross-system entry.
- **Deep-dived two prior 15m builds:**
  - `research/adapted/trader-retro-15m.md` - TS swing bot, monolithic, InMemory, signals on BTC spot.
  - `research/adapted/kalshi-15m-bot-15m.md` - Python multi-strategy, modular, WS feed, RiskGuard, OrderManager.
- **Verdict:** kalshi-15m-bot is the primary donor codebase for KXBTC15M. Both share fatal flaw: no persistent state, no exchange reconciliation.
- **Mystic Bot analysis complete (CORRECTED):**
  - `research/mystic/mystic-performance-summary.md` v2 - Directional bias detection edge analysis.
  - Mystic live data (April 12-15, 2026): 134W/3L, ~97.8% win rate on real Kalshi fills.
  - Assessment: Credible proof-of-concept using directional momentum on over/under contracts.
- **Strategy v3 deployed (CORRECTED):**
  - `docs/STRATEGY-KXBTC15M.md` v3 with correct over/under model: directional entry signals, vol regime classification, risk guardrails, execution FSM, backtesting requirements, deployment path.
- **Performance governance:**
  - `docs/PERFORMANCE-RECORDS.md` with PERF-001 (Mystic data), verification policy, minimum sample sizes.
- **AI Guardrail deployed:**
  - `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md` - Prevents AI from hallucinating range/wedge mechanics.
- No code in zerotrading repo yet. No live deploy.

## Key findings from prior builds

1. Both bots use InMemoryStorage - restart = orphaned Kalshi positions.
2. P&L computed locally from bid prices, not actual fill prices. Fees not accounted.
3. trader-retro signals on BTC spot (wrong instrument). kalshi-15m-bot signals on Kalshi orderbook (correct).
4. kalshi-15m-bot has proper modular architecture (client/feed/orders/risk/strategies) worth porting.
5. Three critical fixes for any port: persistent state, fill verification + fee accounting, exchange reconciliation loop.

## Key findings from Mystic analysis

1. KXBTC15M is over/under, not range. The edge is in predicting direction, not range containment.
2. ~98% win rate suggests Mystic trades selectively - likely enters only when momentum is clear.
3. Fee drag is significant: 7% winner fee + exchange fees on cheap contracts.
4. Tail risk: reversals within 15m windows can wipe streaks.
5. Minimum 500+ trades needed before any claims of edge persistence.

## Decisions pending (user input needed)

1. **Language:** Python (recommended, kalshi-15m-bot is Python) vs TypeScript.
2. **Starting strategy parameters:** Use STRATEGY-KXBTC15M.md v3 defaults, or customize?
3. **More builds to review?** User mentioned there are more.
4. **Port approach:** Copy kalshi-15m-bot src/ structure and add Supabase, or clean rewrite guided by strategy v3?

## Next concrete step

1. User confirms language + starting strategy + whether more builds to review.
2. Port kalshi-15m-bot modular structure into `src/` in this repo.
3. Add Supabase persistence layer.
4. Add boot reconciliation (fetch open Kalshi positions, rebuild local state).
5. Implement directional momentum strategy per STRATEGY-KXBTC15M.md v3.
6. Start in paper mode. Log all trades to Supabase.
7. After 200+ paper trades, evaluate for safe-live mode.

## Do not regress

- **KXBTC15M is over/under, not range.** See `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md`.
- Execution FSM (see `docs/pdfs/zerotrading-core-execution-statemachine.pdf`).
- Accounting / fee corrections (see `docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf`).
- OPSEC: no secrets, no account IDs in repo.
- Mode discipline: paper -> safe-live -> live.
- Performance verification policy: no go-live without exchange-reconciled, fee-adjusted PnL data.
