# alpha-bot — Adapted Research

**Source:** https://github.com/meszaroszack/alpha-bot
**Language:** JavaScript (Node.js)
**Market:** KXBTC15M (15-minute BTC over/under)
**Commits:** 22 | **Deployments:** 18 (Railway)
**Status:** Inactive (last commit ~1 month ago, strategy aligned to KXBTC15M reference-price)

## What It Does

Full-stack trading bot with 4 pluggable strategies, Supabase persistence, Coinbase price polling, OHLCV candle building, and a frontend dashboard. Most mature non-AI algorithmic build.

## Architecture

- `backend/botEngine.js` — Main loop: polls Coinbase, builds candles, runs indicators, places trades
- `backend/indicators.js` — 4 strategy implementations + technical indicator helpers
- `backend/kalshiAuth.js` — Kalshi API auth (RSA key signing)
- `backend/supabase.js` — Persistence layer (bot_runs, signals, trades, logs)
- `frontend/` — Dashboard UI
- `Desktop/cursor-kalshi-dashboard/` — Research dashboard (separate app)

## Strategy Logic

### Four Strategies

| Strategy | Signal Source | Min Confidence | Key Idea |
|----------|-------------|----------------|----------|
| Swing | EMA(5/20) crossover + velocity | 55 | Trend-following via EMA alignment |
| Theta | Time-decay + RSI + momentum | 55 | Sell NO when BTC flat/weak near expiry |
| Scalper | Bollinger Bands(20,2) + ATR | 60 | Mean reversion on BB extremes |
| Momentum | RSI + MACD crossover + divergence | 58 | Directional momentum with confirmation |

### Reference Price Alignment (ALL strategies)
Every strategy adjusts confidence based on BTC vs. market reference price:
- `gapPct = (currentPrice - referencePrice) / referencePrice * 100`
- Bullish/YES: +8 confidence if price above reference, -10 if fighting negative gap
- Bearish/NO: +8 confidence if price below reference, -10 if fighting positive gap

### Risk Management
- `riskPct: 25` (% of balance per trade)
- `maxPositions: 3`
- `dailyLossLimitPct: 20`
- `maxTradeSize: $50`
- `maxContractsPerTrade: 10`
- `cooldownMinutes: 5` (after a loss)
- `stopLossPct: 40` | `takeProfitPct: 50`
- `BALANCE_FLOOR: $5` (stop trading below)
- `MAX_CONSECUTIVE_LOSSES: 3`

### Technical Indicators
SMA, EMA, EMA arrays, RSI(14), MACD(12/26/9), Bollinger Bands(20,2), ATR(14)

## What's Reusable
- **Reference price alignment** — Critical for KXBTC15M, adjusts all signals to the actual binary question
- **Supabase integration pattern** — bot_runs, signals, trades tables
- **4-strategy plugin architecture** — Clean separation, each returns standardized signal object
- **Risk management config** — Daily loss limit, consecutive loss tracking, balance floor
- **Coinbase price polling** — Simple, reliable BTC price source

## What's Wrong
- **No probability model:** Uses technical indicators designed for continuous price instruments, not binary contracts
- **No edge calculation:** Never compares model probability to market-implied probability
- **No orderbook awareness:** Places taker orders without checking spread or depth
- **No fee accounting in signals:** Fee impact not considered in confidence threshold
- **25% risk per trade is extremely high** for a binary that can go to zero
- **Technical indicators (RSI/MACD/BB) are dubious** on 15-min candles for a binary outcome

## Port Recommendations for ZeroTrading
- Port the reference price alignment logic — this is the most valuable piece
- Port the Supabase persistence pattern (schema + insert/update functions)
- Port the standardized signal interface: `{ signal, confidence, reasoning[], stopLoss, targetPrice, strategyName, indicators{} }`
- Replace indicator-based signals with probability-edge model
- Keep risk management framework but reduce per-trade risk to 2-5%
