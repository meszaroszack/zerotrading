# Architecture Diagrams

## Guidelines

This folder holds architecture diagrams, flow diagrams, user journeys, and any visual artifacts that document ZeroTrading's design.

Diagrams here are **canonical references** — agents and humans should treat them as authoritative when they describe system behavior, lifecycle, or boundaries. If a diagram and a markdown doc disagree, open a PR to reconcile them rather than letting drift continue.

## What belongs here

- System architecture diagrams (overall platform, ZeroTrading Core shell, strategy layer, monitoring layer)
- Execution state machine visualizations
- Accounting and reconciliation flow diagrams
- Strategy decision flow diagrams (KXBTC15M Mystic-style, BTC hourly, weather, arbitrage)
- User journeys (operator, researcher)
- Data flow diagrams (ingestion -> archive -> analysis -> dashboard)
- Deployment topology (Railway + Supabase)
- Bubble maps and analytics visualizations for market behavior

## What does NOT belong here

- Screenshots of the live dashboard (use `docs/screenshots/` if needed)
- Sensitive infrastructure topology that exposes attack surface
- Anything containing real API keys, account IDs, or credentials

## File conventions

- Prefer `.svg` for vector diagrams (scales, diffs cleanly)
- `.png` acceptable for raster exports
- `.drawio`, `.excalidraw`, or `.fig` source files welcome alongside exports
- Mermaid diagrams should live inline in markdown docs (not here)
- Use kebab-case filenames: `execution-state-machine.svg`, `kxbtc15m-decision-flow.svg`

## Naming pattern

```
<scope>-<subject>-<version>.<ext>

Examples:
  core-architecture-v1.svg
  core-execution-statemachine-v2.svg
  strategy-kxbtc15m-decision-flow-v1.svg
  monitoring-data-pipeline-v1.svg
  user-journey-operator-v1.svg
```

## Index

When you add a diagram, append it here:

| File | Scope | Description | Last updated |
|---|---|---|---|
| _(none yet)_ | | | |

## Source of truth

These diagrams pair with the authoritative PDFs in `docs/pdfs/`:

- `zerotrading-core-architecture.pdf`
- `zerotrading-core-execution-statemachine.pdf`
- `zerotrading-core-accounting-fee-corrections.pdf`

When updating a diagram, cross-check against the relevant PDF and note any divergences in `ai/summaries/DECISION-LOG.md`.
