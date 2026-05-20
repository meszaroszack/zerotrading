# System Synthesis

Append-only living document. How the ZeroTrading systems fit together and where each interaction's thinking lands.

Each session appends a dated section. Do not rewrite past sections - supersede with a new one.

## Systems in scope
- **KXBTC15M** (flagship; Mystic-style launch wedge on Kalshi BTC 15-min)
- **BTC hourly** (secondary; existing models being hardened)
- **Weather markets**
- **Sports / event systems**
- **24/7 market intelligence layer** (cross-cutting; feeds all of the above)
- **Execution FSM** (shared core)
- **Accounting / fee correction** (shared core)

## Session entry template
```
## YYYY-MM-DD HH:MM ET - <session title>

**By:** <agent / human>
**Linked DECISION-LOG:** <link or `pending`>
**Linked notes:** research/notes/..., research/adapted/..., research/mystic/...

### What changed in our mental model
- ...

### How systems now interact
- KXBTC15M <-> 24/7 monitoring: ...
- KXBTC15M <-> accounting: ...
- BTC hourly <-> KXBTC15M: ...
- Weather <-> sports <-> shared FSM: ...

### What this implies for next build
- ...

### Open contradictions
- ...
```

---

## 2026-05-19 23:00 ET - Initial synthesis (scaffold seeded)

**By:** Comet (browser agent) on behalf of meszaroszack
**Linked DECISION-LOG:** pending
**Linked notes:** none yet

### What changed in our mental model
- ZeroTrading is being built as a multi-system platform with one shared execution FSM and one shared accounting core, not a single-strategy bot.
- Mystic is the *reference design* for KXBTC15M, not the deploy target. We port the wedge pattern, not the codebase.
- The 24/7 monitoring layer is a first-class system, not a side feature. It feeds every strategy with snapshots, anomaly signals, and post-hoc replay data.

### How systems now interact
- KXBTC15M reads from the 24/7 layer's live Kalshi snapshots; writes its own decisions back as JSONL into `research/logs/`.
- All strategies share the FSM (`docs/pdfs/zerotrading-core-execution-statemachine.pdf`) - new strategies plug in by implementing the entry / exit / size hooks.
- Accounting (`docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf`) is the single source of P&L truth - strategies must not maintain their own books.
- Weather and sports inherit the same plumbing but use different feed adapters in the 24/7 layer.

### What this implies for next build
- KXBTC15M should be implemented as a strategy module that consumes the 24/7 feed and emits FSM actions - not a standalone process.
- 24/7 layer needs a minimal v0 (Kalshi orderbook snapshot writer to `research/logs/`) before KXBTC15M can be tested end-to-end.

### Open contradictions
- None yet - this is the seed entry.
