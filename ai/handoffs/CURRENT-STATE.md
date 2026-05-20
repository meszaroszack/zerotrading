# CURRENT STATE

_Live pointer file. Overwrite at end of every session. Append-only history lives in `ai/summaries/DECISION-LOG.md`._

**Last updated:** 2026-05-20 00:00 ET
**Updated by:** Comet (browser agent) on behalf of meszaroszack

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
- No code in zerotrading repo yet. No live deploy.

## Key findings from prior builds
1. Both bots use InMemoryStorage - restart = orphaned Kalshi positions.
2. P&L computed locally from bid prices, not actual fill prices. Fees not accounted. This causes the accounting discrepancy vs Kalshi account.
3. trader-retro signals on BTC spot (wrong instrument). kalshi-15m-bot signals on Kalshi orderbook (correct).
4. kalshi-15m-bot has proper modular architecture (client/feed/orders/risk/strategies) worth porting.
5. Three critical fixes for any port: persistent state, fill verification + fee accounting, exchange reconciliation loop.

## Decisions pending (user input needed)
1. **Language:** Python (recommended, kalshi-15m-bot is Python) vs TypeScript.
2. **Starting strategy:** Simple threshold/last90 first, momentum later? Or start with momentum?
3. **More builds to review?** User mentioned there are more.
4. **Port approach:** Copy kalshi-15m-bot src/ structure and add Supabase, or clean rewrite guided by the analysis?

## Next concrete step
1. User confirms language + starting strategy + whether more builds to review.
2. Port kalshi-15m-bot modular structure into `src/` in this repo.
3. Add Supabase persistence layer.
4. Add boot reconciliation (fetch open Kalshi positions, rebuild local state).
5. Start in paper mode.

## Do not regress
- Execution FSM (see `docs/pdfs/zerotrading-core-execution-statemachine.pdf`).
- Accounting / fee corrections (see `docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf`).
- OPSEC: no secrets, no account IDs in repo.
- Mode discipline: paper -> safe-live -> live.
