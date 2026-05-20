# ZeroTrading Core Architecture (Index)

Authoritative design lives in `docs/pdfs/`:
- `zerotrading-core-architecture.pdf`
- `zerotrading-core-execution-statemachine.pdf`
- `zerotrading-core-accounting-fee-corrections.pdf`

## Layers
1. Market Intelligence (24/7) -> `research/logs/`
2. Strategy -> `docs/STRATEGY-*.md`
3. Execution FSM (restart-safe, reconciled)
4. Accounting (fees, slippage, corrections, P&L)
5. Ops -> `ops/runbooks/`
6. Governance / AI -> `ai/`

## Modes
`paper` -> `safe-live` -> `live`. Gates documented in `ops/runbooks/RAILWAY-DEPLOYMENT.md`.

## Diagrams
See `docs/diagrams/`.
