
# ZeroTrading

ZeroTrading is an AI-first, open-knowledge prediction market platform focused on Kalshi BTC 15-minute (KXBTC15M) markets, BTC hourly markets, weather, and sports/event systems.

This repo is designed as a hybrid of modern AI-agent best practices and traditional SaaS / GitHub structure. It assumes deployment on Railway with Supabase for persistence and analytics.

## What is this?

ZeroTrading is not just a trading bot. It is a **prediction-market operating system** with:

- A hardened execution and accounting shell (ZeroTrading Core)
- Pluggable strategy modules (KXBTC15M Mystic-style first, BTC hourly, weather, sports)
- A 24/7 market monitoring and anomaly detection layer
- An open-knowledge research archive — algorithms, architecture, and methods are fully public
- An AI-agent-first contribution model where every session must update this repo

## Quick Start

> Deployment: Railway + Supabase  
> See `ops/runbooks/RAILWAY-DEPLOYMENT.md` for setup

## For AI Agents

If you are an AI agent (Perplexity, Cursor, Claude Code, or any other), read these before touching any code:

1. `ai/system/README-AI-SYSTEM.md`
2. `ai/system/MODEL-SPEC.md`
3. `ai/system/OPEN-OPSEC-POLICY.md`
4. `docs/ARCHITECTURE-ZEROTRADING-CORE.md`

Then copy and use the master prompt at `ai/prompts/MASTER-CROSS-MODEL-PROMPT.md`.

**Every agent session must update this repo** — decision log, fresh session summary, and relevant docs.

## For Humans

- Strategy rationale and design: `docs/`
- Architecture: `docs/ARCHITECTURE-ZEROTRADING-CORE.md`
- Contribution guide: `CONTRIBUTING.md`
- Deployment: `ops/runbooks/RAILWAY-DEPLOYMENT.md`
- Research: `research/`

## Repo Structure

```
/
├─ .cursor/rules/         # Cursor AI agent rules
├─ ai/
│  ├─ system/             # Canonical AI agent specs and policies
│  ├─ prompts/            # Master prompts for all AI tools
│  ├─ handoffs/           # Session handoff templates
│  ├─ checklists/         # Non-negotiable output contracts
│  ├─ specs/              # Strategy and feature specs
│  ├─ evals/              # Evaluation test cases
│  ├─ summaries/          # Decision log and session summaries
│  └─ schemas/            # JSON schemas for structured agent output
├─ docs/
│  ├─ pdfs/               # Authoritative architecture and design PDFs
│  ├─ ARCHITECTURE-ZEROTRADING-CORE.md
│  ├─ STRATEGY-KXBTC15M.md
│  ├─ STRATEGY-BTC-HOURLY.md
│  └─ MONITORING-OVERVIEW.md
├─ ops/
│  └─ runbooks/           # Deployment and operational runbooks
├─ research/
│  ├─ logs/               # JSONL market behavior logs
│  └─ MONITORING-OVERVIEW.md
├─ src/                   # Application source code
├─ CONTRIBUTING.md
├─ LICENSE
└─ SECURITY.md
```

## Principles

- **Documentation and approval first** — design is documented before implementation
- **Open knowledge** — no hidden algorithm files, no README theater
- **Operational security** — secrets, credentials, and live infra details are never committed
- **AI-agent first** — prompts, rules, handoffs, evals, and schemas are first-class repo citizens
- **Exchange truth wins** — reconciliation to Kalshi is the authoritative source for positions and P&L

## Status

> Active development. See `ai/summaries/DECISION-LOG.md` for latest decisions and progress.
