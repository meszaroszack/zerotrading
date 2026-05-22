# Changelog

All notable changes to ZeroTrading are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

### Added — 2026-05-21 21:16 ET (Phase 1 Python Skeleton — Cross-Repo Sync)
- **`docs/ARCHITECTURE-KXBTC15M.md`** (NEW) — Full implementation architecture for zerotradingx15minbtc Phase 1. Covers 9-layer diagram, file map, FSM state machine, boot sequence, probability model, skip gates, fee formula, vol regime classification, paper vs live mode comparison, all env vars, and Phase 2 scope.
- **`research/adapted/phase1-implementation-patterns.md`** (NEW) — 10 explicit design decisions from the Phase 1 skeleton build: zero-drift BS, taker fee model, Supabase FSM, duplicate-trade guard, in-memory cooldown rationale, async REST class, 30s warmup, mutable btc_spot list, feed health gating, skip gate ordering. Patterns inherited and rejected from prior builds documented.
- **`ai/summaries/DECISION-LOG.md`** — Appended 9 architectural decisions from Phase 1 (cross-repo sync entry with full rationale and related files).
- **`ai/handoffs/CURRENT-STATE.md`** — Updated to reflect beta Phase 1 complete, manual deploy steps, Phase 2 work items.
- **`ai/summaries/2026-05-21-2100-phase1-cross-repo-sync.md`** — Session summary for cross-repo sync session.

### Added — 2026-05-21 (Beta Repo Initialization)
- **Beta repo initialized:** [meszaroszack/zerotradingx15minbtc](https://github.com/meszaroszack/zerotradingx15minbtc) — focused KXBTC15M beta product. Phase 0 (documentation scaffold) complete.
- Inheritance model defined: beta inherits guardrails, strategy spec, architecture principles from this parent repo. Bi-directional contribution pattern established.
- Beta repo structure: 31 files, full documentation baseline (docs/, ai/, ops/, research/, .cursor/rules/). No implementation code yet.
- Beta DECISION-LOG initialized with two entries (scaffold decision, inheritance model).
- Beta session summary committed at `ai/summaries/2026-05-21-2000-init-scaffold.md` in beta repo.

### Added (prior)
- AI-first repo structure: `ai/system`, `ai/prompts`, `ai/handoffs`, `ai/checklists`, `ai/evals`, `ai/schemas`, `ai/summaries`
- Ops runbooks: `ops/runbooks/RAILWAY-DEPLOYMENT.md`, `ops/runbooks/SUPABASE-SETUP.md`
- Research structure: `research/logs/`, `research/MONITORING-OVERVIEW.md`
- Docs: architecture, strategy stubs for KXBTC15M, BTC hourly, weather, sports
- SECURITY.md, CONTRIBUTING.md, LICENSE
- Master cross-agent prompt at `ai/prompts/MASTER-PROMPT.md`
- Session output contract and fresh-session template
- Decision log initialized
