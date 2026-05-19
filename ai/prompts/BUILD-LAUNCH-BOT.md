# Prompt: Build Mystic-Style KXBTC15M Launch Bot

Use this prompt for implementation passes focused on building or refactoring the KXBTC15M strategy module.

---

```
You are building the KXBTC15M launch strategy for ZeroTrading.
Read ai/system/MODEL-SPEC.md and docs/ARCHITECTURE-ZEROTRADING-CORE.md before proceeding.

OBJECTIVE:
Implement a Mystic-inspired KXBTC15M late-window strategy module that plugs into
the existing ZeroTrading Core execution shell. Start with paper mode only.

STRATEGY PARAMETERS (Mystic-style):
- Entry window: final 10-15 minutes before market close
- Price band filter: BTC spot must be within X% of strike
- RSI filter: momentum confirmation
- Volatility cap: skip if recent volatility exceeds threshold
- Cooldown: no re-entry within N minutes of last exit
- Balance-tier risk sizing: Kelly-influenced, balance-aware
- Maker-only pricing: never cross spread on entry
- Cheap-contract exit: price-zone-aware, not noise-reactive

NON-NEGOTIABLES:
- Strategy module must NOT touch accounting, reconciliation, or state machine internals
- Paper mode must work end-to-end before any live behavior
- Every no-trade must log a skip reason
- Every candidate must log a score and evaluation context
- Restart-safe: strategy must restore cleanly from persisted state

DELIVERABLES:
- Strategy module code
- Unit tests or eval cases in ai/evals/
- docs/STRATEGY-KXBTC15M.md with design rationale
- DECISION-LOG entry
- Fresh-session summary pushed to GitHub
```
