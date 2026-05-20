# trader-retro - KXBTC15M Swing Bot (TypeScript)

**Source URL:** https://github.com/meszaroszack/trader-retro
**Live (broken):** https://trader-retro-production.up.railway.app/#/
**Language:** TypeScript (Node/Express + Vite client)
**Pulled by:** Comet on 2026-05-19

## What it does
Single-file trading engine (`server/tradingEngine.ts`) that polls Kalshi KXBTC15M markets every 5s, generates signals via RSI/MACD/swing-detection indicators (`server/indicators.ts`), and enters/exits swing trades with pre-calculated stop-loss and profit-target prices. Uses SSE to push state to a retro-styled React dashboard.

## Architecture
- `server/tradingEngine.ts` - monolithic engine: poll cycle, signal generation, swing entry/exit, state broadcast
- `server/indicators.ts` - RSI(14), EMA, MACD(12/26/9), swing detection (lookback + threshold %), velocity confirmation
- `server/kalshi.ts` - Kalshi API v2 client (REST, RSA-PSS auth, prod + demo endpoints)
- `server/storage.ts` - InMemoryStorage only (no persistence across restarts)
- `client/` - React/Vite dashboard with retro theme
- `shared/schema.ts` - Drizzle schema (trades, signals, settings, credentials)

## What is reusable for ZeroTrading
- **Kalshi API client pattern:** RSA-PSS signature, dual prod/demo endpoints, typed KalshiMarket/KalshiOrder interfaces. Nearly identical to what we need.
- **Swing detection idea:** lookback-N-ticks + threshold-% + velocity confirmation is a valid cheap signal. Needs calibration but the structure is sound.
- **Pre-calculated stop/target at entry time:** avoids price-check overshoot. Good pattern to keep.
- **Market filtering:** sorting by close_time, filtering for KXBTC15M prefix + status open + future close_time. Reusable.
- **SSE broadcast for dashboard state:** lightweight, works with Railway.

## What is wrong / not portable
- **InMemoryStorage:** all state lost on restart. No reconciliation possible. This is the #1 problem - if the process restarts mid-trade, the position is orphaned on Kalshi with no local record.
- **No fill verification:** `placeOrder` returns an order_id but never checks if the order actually filled. The bot assumes filled immediately. On a limit order this means phantom positions.
- **Accounting drift:** P&L calculated from bid prices at exit time, not from actual fill prices. Settlement reconciliation fetches `settled_positions` but only for ticker-closed markets, not for active exits. This explains the discrepancy vs Kalshi account.
- **Signal on BTC spot price, not on Kalshi orderbook:** RSI/MACD/swing are computed on Coinbase BTC spot price history, but the actual Kalshi KXBTC15M contract prices (yes_bid/yes_ask) are separate instruments. Spot may move but the contract may not follow 1:1.
- **Single active trade at a time:** `activeSwingTrade` is a single slot. If the bot misses an exit (restart, error), it cannot track the orphaned position.
- **No daily loss cap enforcement server-side:** `targetBalance` cap pauses the bot on the high side but there is no circuit-breaker on the loss side.
- **25% risk per trade:** default `riskPercent: 25` is extremely aggressive for a 15-min binary.
- **swingThreshold 0.05% on BTC:** at $100k BTC, 0.05% = $50. This triggers on noise. Needs much higher threshold or different signal source.

## Port plan
- **Keep:** Kalshi API client shape (port to Python or TS in ZeroTrading `src/client/`), pre-calculated stop/target pattern, market filtering logic.
- **Rewrite:** Storage must be Supabase-backed. Engine must be restart-safe (reconcile open positions on boot). Signal must incorporate Kalshi orderbook, not just BTC spot. Fill verification must poll order status before recording a position.
- **Drop:** InMemoryStorage, 25% risk default, swing-on-spot-only signal, single-trade-slot architecture.
- DECISION-LOG entry: pending

## Open questions
- Was this bot ever profitable net of fees? User reports positive P&L locally but accounting mismatch vs Kalshi. Need to cross-reference Kalshi trade history.
- The retro dashboard is nice for UX - worth porting the SSE pattern or switch to websocket for ZeroTrading dashboard?
