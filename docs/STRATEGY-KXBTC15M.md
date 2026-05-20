# Strategy: KXBTC15M (Flagship)

Mystic-style launch wedge on Kalshi BTC 15-minute markets.

## Inputs
- Coinbase BTC spot reference
- Kalshi KXBTC15M orderbook + trades
- Time-to-expiry, implied probability surface

## Decision
- Entry window: first N seconds after market launch
- Edge threshold vs implied prob
- Size capped by per-market and per-account exposure rules

## Exit
- Time stop at T-X to expiry
- Adverse-edge stop
- Expiry reconciliation

## Risk
Daily loss cap, per-market cap, kill-switch integrated.
Reasons logged to `research/logs/`.

## Evals
`ai/evals/strategy-decision-cases.yaml`
