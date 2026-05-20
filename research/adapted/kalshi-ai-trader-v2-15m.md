# kalshi-ai-trader-v2 — Adapted Research

**Source:** https://github.com/meszaroszack/kalshi-ai-trader-v2
**Language:** TypeScript
**Market:** KXBTC15M (15-minute BTC over/under)
**Commits:** 12 | **Deployments:** 12 (Railway)
**Status:** Inactive (2 months old)

## What It Does

AI-driven 15-minute BTC binary trading engine with regime detection and LLM-based decision making. Uses Claude/GPT as the decision engine via a structured SYSTEM_PROMPT, not pure algorithmic rules.

## Architecture

- `server/aiEngine.ts` — Core engine: state machine, AI prompt construction, order lifecycle
- `server/kalshi.ts` — Kalshi API layer (REST auth, market discovery, order placement)
- `server/memory.ts` — In-memory state (no persistence)
- `server/storage.ts` — File-based storage
- `client/` — Dashboard UI

## Strategy Logic

### Regime Detection
Five market regimes: `GRINDING_UP`, `GRINDING_DOWN`, `VOLATILE`, `FLAT_NEAR_STRIKE`, `BREAKOUT`

### Entry Conditions
- **Momentum Scalping:** delta_from_open > $60, consistent tick direction, contract 28-72c, >180s remaining
- **Penny Hunting:** 1-8c contracts when BTC shows early momentum toward strike
- **Sizing:** 5% balance standard, 8% high conviction (>0.8), 2% after 3 losses
- **No Trend Fighting:** delta direction must align with price trends
- **Time Gate:** No entries if <90s remaining

### Exit Rules (priority order)
1. Up 35% -> exit immediately
2. Up 20% -> exit if >120s remaining
3. Crossed strike -> exit if held for 2 ticks
4. Down 40% -> exit if >180s remaining
5. <90s remaining and contract <75c -> exit
6. Contract >80c with <2min left -> hold to settlement ($1.00)

### AI Decision Interface
```
AIDecision {
  action: "buy_yes" | "buy_no" | "skip"
  confidence: 0-100
  conviction: 0-1
  model_probability_yes: number
  edge: number
  regime: Regime
  hold_to_settlement: boolean
  size_multiplier: 0.5 | 1.0 | 1.5
}
```

## What's Reusable
- Regime detection concept (adapt for pure algorithmic version)
- Exit priority cascade structure
- Penny hunting logic for cheap contracts
- Conviction-based sizing multiplier
- Hold-to-settlement logic when contract >80c near expiry

## What's Wrong
- **AI dependency:** Uses LLM calls for every decision — adds latency, cost, and unpredictability
- **No persistence:** In-memory only — Railway restarts orphan positions
- **No exchange reconciliation:** Local P&L, never verified against Kalshi
- **No fee accounting:** Kalshi taker fees not subtracted from P&L
- **Cooldown after 5 losses is too aggressive** for a 15-min market (misses 30 min of opportunities)

## Port Recommendations for ZeroTrading
- Port the exit cascade as a pure algorithmic module (no AI needed)
- Port regime detection as a helper function feeding into edge calculation
- Replace AI decision-making with probability-edge model from kalshi-btcd-trader
- Add Supabase persistence from day one
