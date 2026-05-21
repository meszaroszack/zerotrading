# Pre-Code Review Checklist

> Run this checklist before committing ANY code that touches trading, execution, state management, or accounting.
> If any item fails, the code is NOT ready.

## 1. Persistence (BLOCKING)

- [ ] All position state is stored in Supabase, not in-memory
- [ ] All trade records are written to Supabase on execution
- [ ] All signal/decision records are written to Supabase
- [ ] No module-level or global state that would be lost on process restart
- [ ] State is read from Supabase on boot, not from local cache

## 2. Exchange Reconciliation (BLOCKING)

- [ ] On startup: fetch all open positions from Kalshi API
- [ ] On startup: fetch current Kalshi balance
- [ ] On startup: compare Kalshi positions against Supabase records
- [ ] On startup: log all reconciliation results (matches, mismatches, orphans)
- [ ] On startup: resolve discrepancies before entering trading loop
- [ ] The exchange is treated as the source of truth, not local state

## 3. P&L Accuracy (BLOCKING)

- [ ] P&L computed from actual Kalshi fill prices (not bid/ask at decision time)
- [ ] Kalshi fees subtracted using actual formula: `0.07 * contracts * price * (1-price)`
- [ ] Settlement P&L = actual payout - actual entry cost - actual fees
- [ ] Periodic balance reconciliation against Kalshi balance

## 4. Market Structure (BLOCKING)

- [ ] Code correctly identifies which market it targets (KXBTC15M vs KXBTCD)
- [ ] No strategy confusion between 15-min binary and hourly strike ladder
- [ ] Settlement logic matches market type (60-sec CFB RTI average for both)
- [ ] Fee calculations use correct Kalshi formula

## 5. Execution Safety

- [ ] Paper mode available and tested before live
- [ ] Circuit breakers / loss limits implemented
- [ ] Order status polling with timeout (no phantom positions)
- [ ] Unfilled orders cancelled within reasonable time
- [ ] Balance floor respected (stop trading below minimum)

## 6. Strategy Alignment

- [ ] Strategy matches `docs/STRATEGY-KXBTC15M.md` or `docs/STRATEGY-BTC-HOURLY.md`
- [ ] Entry/exit rules match documented spec
- [ ] No undocumented strategy changes (document first, then code)

## References

- `ai/guardrails/PERSISTENCE-RECONCILIATION.md` (detailed rules + error history)
- `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md` (market structure)
- `docs/STRATEGY-KXBTC15M.md` (strategy spec)
- `docs/pdfs/zerotrading-core-execution-statemachine.pdf` (FSM design)
- `docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf` (accounting)
