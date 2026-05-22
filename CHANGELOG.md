# Changelog

All notable changes to ZeroTrading are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]


### Added — 2026-05-21 21:33 ET (Core Stabilization + Governance Integration)
- **zerotrading-core governance scaffold** — `ai/` directory, `.cursor/rules/`, `CHANGELOG.md`, `CONTRIBUTING.md`, `SECURITY.md`, `README.md` added to zerotrading-core. Full knowledge transfer from this parent repo.
- **`ai/summaries/2026-05-21-2133-core-stabilization-session.md`** — Session summary for Core stabilization session (11 bug fixes).
- **`ai/summaries/DECISION-LOG.md`** — Appended Core session entry with all 12 decisions.
- **`ai/handoffs/CURRENT-STATE.md`** — Updated to reflect Core + 15M status.

### Fixed — 2026-05-21 (ZeroTrading Core, 11 bugs)
- **I-3 crash loop** (`78a2055`) — ENTERING+null activeOrder → MANUAL_REVIEW
- **Railway TS2353** (`e9ab458`) — proximityPenaltyBps type
- **I-9 invariant** (`485fef1`) — PERSIST_CONTEXT after order transitions
- **Fill race + [object Object]** (`826a949`) — effect ordering + Dashboard serialization
- **Time-exit removed** (`be99ad2`) — operator decision: do not sell because market is ending
- **count_fp root cause** (`62f2126`) — normalizeCountFp() at adapter boundary; root cause of all entry_abandoned_no_fill failures
- **404 on getOrder** (`82b0bd9`) — treat as executed, not error
- **Async RECONCILING** (`0dd95bd`) — 2-min watchdog replaces blocking retry loops
- **Blank trade history** (`e3d224b`) — ACCT_OPEN_POSITION → positionStore.create()
- **Settlement detection** (`d87cfe0`) — GET /markets/{ticker} → status===finalized (Dec 2025 API breaking change)
- **Fill prices + P&L** (`f4b7bdc`) — no/yes_price_dollars, 60-min re-entry guard, exit P&L to store

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
