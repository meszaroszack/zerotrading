# kalshi-btcd-trader — Adapted Research (Hourly, but critical for probability model)

**Source:** https://github.com/meszaroszack/kalshi-btcd-trader
**Language:** TypeScript
**Market:** KXBTCD (hourly BTC strike ladder)
**Commits:** 20 | **Deployments:** 22 (Railway)
**Status:** Active (updated 3 days ago). Most sophisticated quant build.

> **Why this is here:** Although this targets the hourly market, it contains the probability model and trading engine architecture that should be adapted for KXBTC15M. This is the blueprint for ZeroTrading's edge calculation.

## What It Does

Production-grade scored-edge trading system for KXBTCD hourly markets. 5-stage pipeline: candidate evaluation, risk model, selection, execution, reconciliation. Uses terminal probability model (Black-Scholes) with per-minute sigma from Binance 1m candles.

## Architecture

- `server/tradingEngine.ts` — 5-stage pipeline: evaluate, risk, select, execute, reconcile
- `server/touchProb.ts` — Terminal probability model v8.1 (Black-Scholes, normCDF, per-minute sigma)
- `server/marketData.ts` — Binance funding rate, external BTC data
- `server/deribit.ts` — Deribit implied volatility signals
- `server/polymarket.ts` — Polymarket sentiment signals
- `server/kalshi.ts` — Full Kalshi API with orderbook depth, slippage estimation
- `server/storage.ts` — Persistence layer
- `shared/schema.ts` — Trade types
- `tests/` — Test suite

## Probability Model (touchProb.ts v8.1)

### Terminal Distribution (NOT barrier-touch)
- Settlement rule: YES wins if BTC_final > K at expiry
- Model: `p_yes = normCDF(m / sigma_T)` where `m = ln(S/K)` (log-moneyness)
- `sigma_T = sigma_min * sqrt(T_minutes)` (per-minute sigma scaled by sqrt of time)
- `sigma_min` = per-minute stdev of log-returns from last 30 Binance 1m candles
- Typical BTC sigma_min: 0.0005-0.0015 (0.05-0.15% per minute)

### Calibration
- When K = S (at-the-money): p_yes must be 0.5 +/- 0.001
- When |m| = 2 * sigma_T: p_yes must be in [0.02, 0.05]
- Sigma floor: 0.0002 (~29% annualized), Sigma ceil: 0.0050 (~726% annualized)

## Trading Engine (scored_edge_mode)

### 5-Stage Pipeline
1. **Candidate Evaluation:** Score all current-hour markets
2. **Risk Model:** Independent circuit breakers, daily/hourly limits
3. **Selection:** Top-ranked candidate that passes all filters
4. **Execution:** Orderbook-aware, slippage-checked
5. **Reconciliation:** Position accounting, settlement attribution

### Edge Calculation
- `rawEdgeBps = (estimatedNoProbability - impliedNoProbability) * 10000`
- `score = netEdgeBps * rewardRiskRatio * zoneBonus`
- Slippage penalty applied via orderbook depth telemetry

### Regime Detection
Six regimes: `trend_up`, `trend_down`, `mean_reverting`, `breakout_risk`, `high_vol_chop`, `dead_market`

### Circuit Breakers
- `loss_streak`, `stop_loss_cluster`, `hourly_loss`, `daily_loss`
- Can block specific tickers or halt all trading

### Exit Logic
- Zone-aware (cheap/medium/expensive)
- Stop-loss with confirming cycles to reduce noise
- Settlement attribution tracking

## What's Reusable (CRITICAL for KXBTC15M)
- **Terminal probability model** — Adapt for 15-min window (change T from 60 to 15 minutes)
- **Per-minute sigma from Binance 1m candles** — Directly portable
- **Edge calculation: model probability vs market-implied** — The core alpha mechanism
- **Scored ranking across candidates** — Evaluate multiple markets simultaneously
- **Orderbook-aware execution with slippage estimation** — Real execution quality
- **Circuit breakers** — Production-grade risk management
- **Settlement attribution** — Proper P&L tracking
- **Regime detection** — Filter for favorable conditions

## What Needs Adaptation for 15-Min
- T_minutes changes from ~60 to ~15 (sigma_T scaling)
- Strike ladder logic N/A for KXBTC15M (single reference price, not multiple strikes)
- KXBTC15M settles on reference price (over/under), not strike ladder
- Candidate scoring across markets less relevant (only 1 BTC 15-min market active at a time)
- Fee structure identical (0.07 * C * P * (1-P) for takers)

## Port Recommendations for ZeroTrading
- **Port touchProb.ts almost verbatim** — adjust T parameter for 15-min window
- Port edge calculation (model prob vs implied prob from contract price)
- Port circuit breaker framework
- Port orderbook depth / slippage estimation
- Port settlement attribution for proper P&L
- Simplify candidate scoring (single market at a time for 15-min)
