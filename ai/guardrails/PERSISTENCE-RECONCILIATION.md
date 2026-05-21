# GUARDRAIL: Persistence & Exchange Reconciliation

> **REQUIRED READING** for every AI agent writing code in this repository.
> Cross-referenced from: `ai/prompts/MASTER-PROMPT.md`, `ai/handoffs/CURRENT-STATE.md`

## Why This Exists

Every prior trading bot in this portfolio (7 builds, 200+ total commits) failed for the same two reasons:

1. **No persistence** — State lived in-memory or in local files. Railway restarts (which happen regularly) orphaned positions on Kalshi that the bot didn't know about.
2. **No exchange reconciliation** — Local P&L was computed from bid prices at decision time, not from actual fill prices. Kalshi fees were never subtracted. Over time, the dashboard showed profit while the actual Kalshi balance showed loss.

These are not edge cases. They are the #1 and #2 causes of real money loss across all prior builds.

## Non-Negotiable Rules

### Rule 1: ALL state MUST be persisted in Supabase

**NEVER** use any of the following as the primary source of truth:
- In-memory variables, objects, or maps
- Local files (JSON, SQLite, flat files)
- Environment variables for state
- `global` or module-level state that doesn't survive a process restart

**ALWAYS:**
- Write every trade, position, signal, and P&L record to Supabase
- Read state from Supabase on boot (not from local cache)
- Treat the Supabase record as the source of truth, not the in-memory copy

**Test:** Kill the process at any point. Restart it. The bot must resume correctly with zero manual intervention.

### Rule 2: Exchange reconciliation on EVERY boot

On startup, the bot MUST:
1. Fetch all open positions from Kalshi API (`getOpenPositions`)
2. Fetch current Kalshi balance (`getBalance`)
3. Compare Kalshi positions against Supabase positions
4. Reconcile any discrepancies (orphaned positions, missed settlements, balance drift)
5. Log all reconciliation results
6. Only then enter the main trading loop

**NEVER** assume local state matches exchange state. The exchange is always right.

### Rule 3: P&L from actual fills, not estimates

- Query actual fill prices from Kalshi API after order execution
- Subtract actual Kalshi fees (formula: `0.07 * contracts * price * (1 - price)` for takers)
- Settlement P&L = actual settlement payout minus actual entry cost minus actual fees
- Reconcile computed P&L against Kalshi balance delta periodically

**NEVER** compute P&L from the bid/ask price at decision time. That is not the fill price.

### Rule 4: Settlement attribution

- Every trade must be tracked from entry to settlement (or exit)
- Settlement outcomes must be fetched from Kalshi and recorded in Supabase
- Unsettled trades at process restart must be detected and tracked to completion

## Self-Test for AI Agents

Before submitting any code that touches trading, execution, or state management, verify:

| # | Question | Required Answer |
|---|----------|----------------|
| 1 | Where is position state stored? | Supabase (not in-memory) |
| 2 | What happens if the process restarts mid-trade? | Bot fetches open positions from Kalshi, reconciles with Supabase, resumes FSM |
| 3 | Where does P&L come from? | Actual Kalshi fill prices and settlement payouts, minus actual fees |
| 4 | Is there any in-memory-only state that would be lost on restart? | NO (or if yes, it's non-critical cache that gets rebuilt from Supabase/Kalshi) |
| 5 | Does the bot verify its balance matches Kalshi on startup? | YES |
| 6 | Can the bot detect an orphaned position it didn't open? | YES (via Kalshi position fetch on boot) |

If ANY answer is wrong, the code is not ready for review.

## Error History

| Build | Failure | Impact |
|---|---|---|
| trader-retro | In-memory only, no persistence | Orphaned positions on Railway restarts |
| kalshi-15m-bot | In-memory only, no persistence | Same |
| kalshi-ai-trader-v2 | In-memory only, no reconciliation | Dashboard showed profit, Kalshi showed loss |
| alpha-bot | Supabase for logs but not for position state | Partial persistence, still lost state on restart |
| kalshi-trader | File-based storage, no reconciliation | 111 commits never solved the fundamental problem |
| kalshi-btcd-trader | File-based storage | Better architecture but same persistence gap |
| zerotrading-core | References Supabase but implementation incomplete | Correct intent, incomplete execution |

## Related Documents

- `docs/STRATEGY-KXBTC15M.md` §§6-7 (Accounting Requirements, Execution FSM)
- `docs/pdfs/zerotrading-core-execution-statemachine.pdf` (FSM design)
- `docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf` (Fee/P&L truth model)
- `ai/guardrails/KALSHI-MARKET-REFERENCE.md` (Fee structure)
