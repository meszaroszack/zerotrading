# ZeroTrading

> AI-first, open-knowledge prediction market platform built on Kalshi.

ZeroTrading is an open-source operating platform for short-duration prediction markets. The immediate focus is KXBTC15M (Kalshi BTC 15-minute markets), BTC hourly strategies, weather markets, and sports/event systems. The broader mission is to become the most transparent and comprehensive knowledge base for prediction market behavior.

## Architecture

The platform is built around a hardened execution and accounting core (ZeroTrading Core) with pluggable strategy modules. All deployment is via **Railway**, all persistence and analytics via **Supabase**.

```
Ingestion Layer (Binance + Kalshi)
    ↓
Strategy Context Engine
    ↓
Strategy Modules (KXBTC15M | BTC Hourly | Weather | Sports)
    ↓
Risk Engine
    ↓
Execution State Machine ← (protected core)
    ↓
Kalshi Exchange
    ↓
Reconciliation ← (protected core)
    ↓
Accounting + Audit Log ← (protected core)
    ↓
Operator Dashboard + Research / Monitoring Layer
```

See `docs/ARCHITECTURE-ZEROTRADING-CORE.md` for full architecture detail and links to PDF design references.

## If You Are an AI Agent

Read these files first, in order:

1. `ai/system/README-AI-SYSTEM.md`
2. `ai/system/MODEL-SPEC.md`
3. `ai/system/OPEN-OPSEC-POLICY.md`
4. `ai/system/REPO-PRINCIPLES.md`
5. `docs/ARCHITECTURE-ZEROTRADING-CORE.md`

Then consult `ai/prompts/MASTER-PROMPT.md` for the copy-paste operating prompt used across all agents.

Every session must end with an update to `ai/summaries/DECISION-LOG.md` and a fresh-session summary using `ai/handoffs/FRESH-SESSION-TEMPLATE.md`.

## Strategy Priorities

| Priority | Strategy | Status |
|---|---|---|
| 1 | Mystic-style KXBTC15M late-window bot | In development |
| 2 | BTC hourly models (existing, needs accounting fix) | Stabilizing |
| 3 | Adapted models from GitHub, sports, weather | Research phase |
| 4 | 24/7 market monitoring and anomaly analytics | Parallel build |

## Platform Goals

- **Full transparency**: no hidden algorithm files, no README theater.
- **Reproducible research**: every strategy, observation, and finding documented.
- **AI-first workflow**: agents contribute directly to the repo with documentation.
- **Market intelligence**: always-on monitoring of Kalshi markets for anomalies, behavior patterns, and cross-venue signals.
- **Open knowledge**: operate like Wikipedia for prediction market research.

## Deployment

- App: [Railway](https://railway.app)
- Persistence + Analytics: [Supabase](https://supabase.com)
- Live app: https://zerotrading-core-production.up.railway.app

## Contributing

See `CONTRIBUTING.md`. Humans and AI agents follow the same contribution rules.

## License

MIT — see `LICENSE`.
