# CURRENT STATE

_Live pointer file. Overwrite at end of every session. Append-only history lives in `ai/summaries/DECISION-LOG.md`._

**Last updated:** 2026-05-21 ~02:00 ET
**Updated by:** Comet (browser agent) on behalf of meszaroszack

## CRITICAL: Know your market before doing ANYTHING

**`ai/guardrails/KALSHI-MARKET-REFERENCE.md`** is REQUIRED READING.

This repo targets TWO different Kalshi BTC markets. They have DIFFERENT structures:

| System | Market | Structure | Status |
|---|---|---|---|
| ZeroTrading Core | **KXBTCD** (hourly) | Ladder of strike prices; you pick a threshold; above/below that level | **LIVE** on Railway |
| ZeroTrading 15M | **KXBTC15M** (15-min) | Single strike = current price; simple up/down | Paper mode (planned) |

- **KXBTCD** = strike distance, threshold selection, price ladders. Live at `zerotrading-core-production.up.railway.app`.
- **KXBTC15M** = simple binary over/under. No ladders, no strike distance. See `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md`.

**Do NOT mix strategies between markets.** Do NOT apply 15M logic to KXBTCD or vice versa.

## Where we are

### Phase: RESEARCH COMPLETE -> READY TO BUILD

All prior builds have been deep-dived, documented, and committed to `research/adapted/`. The strategy direction is clear. Next step is writing the KXBTC15M strategy spec and building code.

### ZeroTrading Core (KXBTCD - Hourly) - LIVE

- Deployed at `zerotrading-core-production.up.railway.app`.
- Connected to Kalshi exchange (LIVE, real money).
- Balance: ~$4.16 (as of last check).
- Status: CONNECTED, evaluating KXBTCD contracts, making NO_TRADE decisions (strike too close, net edge too low).
- Architecture docs: `docs/pdfs/` (3 canonical PDFs uploaded and verified).

### ZeroTrading 15M (KXBTC15M) - RESEARCH PHASE COMPLETE

**Build Inventory (7 prior builds reviewed):**

| Build | Repo | Lang | Commits | Market | Key Contribution | Research File |
|---|---|---|---|---|---|---|
| trader-retro | meszaroszack/trader-retro | TS | few | 15m | Swing detection, Kalshi API typing | research/adapted/trader-retro-15m.md |
| kalshi-15m-bot | meszaroszack/kalshi-15m-bot | Python | moderate | 15m | Modular architecture, WebSocket, 6 strategies, RiskGuard | research/adapted/kalshi-15m-bot-15m.md |
| kalshi-ai-trader-v2 | meszaroszack/kalshi-ai-trader-v2 | TS | 12 | 15m | Regime detection, AI decision engine, penny hunting | research/adapted/kalshi-ai-trader-v2-15m.md |
| alpha-bot | meszaroszack/alpha-bot | JS | 22 | 15m | 4 strategies, Supabase persistence, reference price alignment | research/adapted/alpha-bot-15m.md |
| kalshi-trader | meszaroszack/kalshi-trader | TS | 111 | 15m | External signals, bracket orders, hour-based bias, decay entries | research/adapted/kalshi-trader-15m.md |
| kalshi-btcd-trader | meszaroszack/kalshi-btcd-trader | TS | 20 | Hourly | **Terminal probability model (Black-Scholes), edge calculation, circuit breakers** | research/adapted/kalshi-btcd-trader-hourly.md |
| Mystic (external) | N/A | N/A | N/A | 15m | 134W/3L performance benchmark, directional momentum | research/mystic/ |

**Stubs/Empty (no code to mine):** kalshi-15m-scalper, Dopealkshi, kalshi-btcd-ai-trader (1-commit snapshot)

**Dashboard/UI only:** compd-trader (Next.js dashboard for alpha-bot), cursor-kalshi-dashboard

### Cross-Build Synthesis: What Actually Matters

Every build shares the same two fatal flaws:
1. **No persistence** — In-memory or file-based. Railway restarts orphan positions.
2. **No exchange reconciliation** — Local P&L never verified against Kalshi.

The builds that got closest to working:
- **kalshi-btcd-trader** has the right architecture (probability model, edge calc, circuit breakers) but targets hourly
- **alpha-bot** has the right infrastructure (Supabase, reference price alignment) but wrong signal source (technical indicators on binaries)
- **kalshi-trader** has the most iteration (111 commits) but never converged — classic AI drift

### Strategy Direction for KXBTC15M

**Core approach: Probability-edge trading (adapted from kalshi-btcd-trader)**
1. Compute model fair value using terminal probability (Black-Scholes, per-minute sigma from Binance 1m candles)
2. Compare model probability to market-implied probability (contract price)
3. Trade when edge exceeds fee + slippage threshold
4. Hold to settlement (natural exit for 15-min binary)

**Supporting signals:**
- Reference price alignment (from alpha-bot)
- Regime detection (from kalshi-btcd-trader)
- External signals: funding rate, OB pressure (from kalshi-trader)
- Hour-based confidence adjustment (from kalshi-trader)

**Infrastructure:**
- Python (ecosystem advantage for quant)
- Supabase persistence (from alpha-bot pattern)
- Boot reconciliation (fetch open positions on restart)
- Paper mode from day one
- Fee-aware accounting

## Next Steps

1. **Write `docs/STRATEGY-KXBTC15M.md` v2** — Full spec based on research synthesis
2. **Build code skeleton** — Python, paper mode, Supabase, probability model
3. **Validate probability model** — Backtest against historical 15-min settlements
4. **Paper trade** — Run against live markets without real money
5. **Go live** — Small balance, monitor, iterate

## Do Not Regress

- KXBTCD and KXBTC15M are DIFFERENT markets
- KXBTC15M is a simple binary over/under, NOT convergence/range/wedge
- Settlement uses 60-second CFB RTI average beginning 1 min BEFORE expiration
- Maker orders have NO fees; settlement has NO fees
- Technical indicators (RSI/MACD/BB) are dubious for 15-min binary outcomes
- The real edge is model probability vs market-implied probability, not pattern recognition
- **ALL state MUST be persisted in Supabase, NEVER in-memory or file-based** (see `ai/guardrails/PERSISTENCE-RECONCILIATION.md`)
- **Every boot MUST reconcile positions and balance with Kalshi exchange** before entering the trading loop
- **P&L MUST come from actual Kalshi fill prices and settlements**, not from bid/ask estimates
- No code that touches trading/execution/state is ready for review until the 6-question self-test in PERSISTENCE-RECONCILIATION.md passes
