# Research Logs — 24/7 Market Intelligence

Append-only logs from the always-on market intelligence layer.

## Scope
- Kalshi orderbook + trade snapshots (KXBTC15M, BTC hourly, weather, sports)
- Reference feeds: Coinbase BTC spot, weather pulls, sports clocks
- Strategy decisions (entries, skips, exits) with reason codes
- Reconciliation diffs (local vs exchange)

## Schema
See `ai/schemas/session-summary.schema.json`. Per-event log lines are JSONL.

## Retention
- Hot: 14 days in Supabase `logs_hot`
- Warm: 90 days in `logs_warm`
- Cold: object storage, compressed JSONL by day

## OPSEC
No API keys, account IDs, or PII committed here. Public summaries only.
