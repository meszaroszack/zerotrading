# Strategy: BTC Hourly (Secondary)

Hourly Kalshi BTC markets; same FSM as flagship.

## Inputs
- BTC spot, realized vol, microstructure features
- Kalshi hourly orderbook

## Decision
- Edge model vs implied prob; threshold-gated entries
- Position sizing per exposure caps

## Exit
- Time stop, adverse-edge stop, expiry reconciliation

## Risk
Inherits global guardrails (`.cursor/rules/trading-safety.mdc`).
