# Changelog

All notable changes to ZeroTrading are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

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
