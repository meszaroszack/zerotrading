# Prompt: Research Archive Builder

Use this prompt to build or extend the 24/7 market monitoring and archive system.

---

```
You are building the ZeroTrading market intelligence and archive layer.
Read research/MONITORING-OVERVIEW.md before proceeding.

OBJECTIVE:
Build or extend a 24/7 monitoring subsystem that continuously records:
- KXBTC15M market snapshots (every evaluation cycle)
- BTC spot and short-horizon candle context
- Spread, top-of-book, depth, and quote changes over time
- Market open/close/settlement timing
- Cross-venue signals: Kalshi vs Polymarket lag/spread events
- Decision engine outputs: all candidates, scores, skip reasons
- Anomaly flags: saturation, extreme spread, late-window swings

DATA DESTINATION:
- Supabase tables (structured)
- research/logs/ JSONL files (machine-readable, for replay/analysis)

ANALYTICS OUTPUT:
- Daily AI-readable anomaly summaries
- Bubble-map and clustering data exports
- Operator-facing analytics graphics

NON-NEGOTIABLES:
- Archive subsystem must not block or affect live trading engine
- Data schema must match ai/schemas/
- Every archive table must have a corresponding schema doc

DELIVERABLES:
- Archive subsystem code
- Supabase table schemas
- research/MONITORING-OVERVIEW.md updated
- DECISION-LOG entry
- Fresh-session summary pushed to GitHub
```
