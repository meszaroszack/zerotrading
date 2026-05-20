# kalshi-trader — Adapted Research

**Source:** https://github.com/meszaroszack/kalshi-trader (private)
**Language:** TypeScript
**Market:** KXBTC15M (15-minute BTC over/under)
**Commits:** 111 | **Deployments:** 77 (Railway)
**Status:** Inactive (2 months old). Most iterated build in the portfolio.

## What It Does

The original Kalshi 15-min trading bot. Massive iteration history (24 "zip" releases). Swing trading with bracket orders, external signal integration, trend detection, and hour-based confidence adjustments.

## Architecture

- `server/tradingEngine.ts` — 781 lines. Core engine: cycle loop, entry/exit logic, bracket order management
- `server/indicators.ts` — Signal generation (RSI, MACD, trend detection)
- `server/externalSignals.ts` — External data: Fear & Greed Index, Funding Rate, OB Pressure
- `server/kalshi.ts` — Kalshi API with bracket order status polling
- `server/storage.ts` — File-based persistence
- `client/` — Dashboard with "price to beat" line, dual charts, compact signals

## Strategy Logic

### Core Approach: Swing Trading with Bracket Orders
- Enter based on directional signal + confidence threshold
- Immediately place bracket orders (take-profit + stop-loss) after entry
- Poll bracket orders every cycle, cancel the loser when winner fills

### Entry Logic
- `generateSignal()` produces direction + confidence + reasoning
- Confidence must exceed `minConfidence` + dynamic `hourDelta` adjustment
- External signals can block entry if opposition exceeds threshold
- Trend opposition applies a penalty to confidence
- Decay entries (late in candle) target 97c contracts

### Exit Logic
- **Bracket Orders:** Target and stop orders placed simultaneously
- **End-of-Round Mode:** Near market close, cancels brackets and rides position to settlement if profitable
- **Order Monitoring:** Polls fill status every cycle, cancels unfilled counterpart

### Hour-Based Bias
- `parseHourBias()` adjusts confidence delta based on time of day
- `getHourConfidenceDelta()` applies hourly modifiers to entry thresholds

### External Signals
- Fear & Greed Index
- Funding Rate (bullish/bearish bias)
- Order Book Pressure
- Each can add/subtract from confidence or block trades entirely

## What's Reusable
- **Bracket order pattern** — Simultaneous TP/SL placement is proper risk management
- **External signal integration** — Funding rate and OB pressure are real alpha sources
- **Hour-based confidence adjustment** — Market behavior varies by time of day
- **End-of-round settlement ride** — Important for binary contracts near expiry
- **Decay entry targeting 97c** — Smart late-window entry for high-probability settlement

## What's Wrong
- **File-based persistence** — No database, fragile on Railway restarts
- **No probability model** — Confidence scores are arbitrary, not calibrated to actual probabilities
- **No edge calculation** — Never compares model fair value to market price
- **No fee awareness** — Fees not factored into entry decisions
- **Bracket orders add complexity without clear benefit** on a 15-min binary where settlement is the natural exit
- **111 commits suggests heavy iteration without convergence** — classic AI drift pattern

## Port Recommendations for ZeroTrading
- Port external signal integration (funding rate, OB pressure)
- Port hour-based confidence adjustment concept
- Port end-of-round settlement logic
- Port decay entry (97c targeting) for late-window opportunities
- Replace bracket orders with simpler hold-to-settlement or time-based exit
- Replace file storage with Supabase
