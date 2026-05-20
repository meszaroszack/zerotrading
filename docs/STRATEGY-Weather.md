# Strategy: Weather Markets

Kalshi weather markets (temperature, precipitation, etc.).

## Inputs
- Multi-source forecast ensemble
- Observation streams near resolution
- Kalshi orderbook

## Decision
- Probability vs implied; threshold + cost-of-edge gating
- Size scaled by forecast confidence

## Exit
- Time-decay near resolution
- Observation-driven re-pricing

## Risk
Per-market cap; daily loss cap; reconciliation at settle.
