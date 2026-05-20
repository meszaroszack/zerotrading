# STRATEGY: KXBTC15M Over/Under v3

> **Version:** 3.0 (Corrected market structure + Mystic-informed)
> **Status:** PAPER MODE - not approved for live trading.
> **Last updated:** 2026-05-20
> **Authoritative references:** See `docs/pdfs/` for execution FSM and accounting.

---

## CRITICAL: KXBTC15M Market Structure

**READ THIS FIRST. Every AI agent working on this repo must internalize this section.**

KXBTC15M is a **simple binary over/under contract.** It is NOT a range, wedge, band, or volatility product.

### How it works

1. At the start of each 15-minute window, the contract sets a **strike price** = BTC spot price at that moment.
2. The contract asks ONE question: **"Will BTC be ABOVE or BELOW this strike price at the end of the 15 minutes?"**
3. You buy **YES** (over) or **NO** (under). That's it.
4. At settlement, if BTC is above the strike, YES pays $1.00, NO pays $0.00. If below, the reverse.

### What it is NOT

- NOT a range/band product (there is no "stays within +/-$250")
- NOT a wedge or convergence product
- NOT a volatility product
- NOT a multi-strike ladder (there is ONE strike per window)
- There are no "brackets" or "strike distances" - just one price, over or under

### Why AI agents get this wrong

AI models frequently hallucinate additional complexity onto binary contracts. Common errors:
- Inventing range/band mechanics that don't exist
- Assuming multiple strike levels per window
- Applying options-style volatility surface thinking
- Confusing KXBTC15M with hourly or daily range products on other platforms

**If any document in this repo references ranges, wedges, bands, convergence zones, or strike distances for KXBTC15M, that document contains an error and must be corrected.**

---

## 1. Strategy Overview

**Thesis:** At the start of each 15-minute window, determine whether BTC is more likely to be above or below the strike at settlement. When the Kalshi orderbook misprices this probability, take the other side.

**Edge source:** Gap between the market's implied probability (from YES/NO prices) and our model's estimated probability of BTC finishing above/below strike.

**Core question every trade answers:** "Is BTC more likely to go up or down in the next 15 minutes, and is the market mispricing this?"

**Inspired by:** Mystic Bot live-money results (April 2026). See `research/mystic/mystic-performance-summary.md`.

---

## 2. Entry Rules

### Pre-conditions (ALL must be true)

1. **Volatility regime = CALM or MODERATE.** (Defined below.)
2. **Spread <= $0.04** on the target contract.
3. **Time filter:** Enter within first 5 minutes of the 15m window (while there's still time for edge to play out).
4. **No scheduled macro events** within the current or next window (FOMC, CPI, NFP, etc.).
5. **Daily loss limit NOT hit.**

### Entry signal

- Read the YES (over) ask price from the Kalshi orderbook.
- Compute model probability of BTC finishing above strike:
  - Inputs: recent momentum (last 5m, 15m, 1h), order flow imbalance, volatility regime.
  - `P_model_over = f(momentum, orderflow, vol_regime)`
- Compute implied probability: `P_implied_over = YES_ask_price / 100`.
- **Buy YES (over) if:** `P_model_over - P_implied_over >= EDGE_THRESHOLD` (default: 0.08).
- **Buy NO (under) if:** `(1 - P_model_over) - (1 - P_implied_over) >= EDGE_THRESHOLD`, i.e., model says under is underpriced.
- **Skip if:** No edge on either side.

### Position sizing

- **Max contracts per trade:** 10 (paper mode) / TBD (live).
- **Max risk per trade:** $50 (paper) / TBD (live).
- **Max concurrent positions:** 1 (single-contract focus initially).

---

## 3. Exit Rules

- **Primary exit:** Hold to settlement. (Contract resolves at end of 15m window.)
- **Early exit (optional):** If momentum sharply reverses against position before T-5min, market-sell to cut losses.
- **No averaging down.** If the position is losing, do not add.

---

## 4. Volatility Regime Classification

| Regime | Definition | Action |
|---|---|---|
| CALM | 1h realized vol < 0.3% | Trade normally |
| MODERATE | 1h realized vol 0.3-0.6% | Trade with tighter edge threshold (+2%) |
| HIGH | 1h realized vol 0.6-1.0% | Reduce position size by 50% |
| EXTREME | 1h realized vol > 1.0% OR scheduled macro event | **NO TRADE** |

Note: In CALM regimes, the over/under is close to 50/50 (slight drift bias). Edge comes from reading short-term momentum correctly. In HIGH regimes, directional moves are larger but less predictable.

---

## 5. Risk Guardrails

| Guardrail | Threshold | Action |
|---|---|---|
| Daily loss limit | -$100 (paper) | Halt all trading for remainder of day |
| Consecutive loss streak | 3 losses in a row | Pause 1 hour, then reassess |
| Weekly loss limit | -$300 (paper) | Halt trading, trigger manual review |
| Max daily trades | 40 | Hard stop, no more entries |
| Spread blowout | Spread > $0.06 | Log warning, do not enter new trades |

---

## 6. Accounting Requirements

**Do not regress from `docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf`.**

Every trade must log:
- Entry timestamp, contract ticker, side (YES/NO), limit price, fill price.
- Exit timestamp, settlement price or sell price.
- Gross PnL = (exit_price - entry_price) * contracts.
- Fees = Kalshi winner fee (7% of profit) + exchange fee ($0.02/contract).
- Net PnL = Gross PnL - Fees.
- Cumulative daily/weekly/all-time net PnL.

All stored in Supabase `trades` table with exchange reconciliation.

---

## 7. Execution FSM

**Do not regress from `docs/pdfs/zerotrading-core-execution-statemachine.pdf`.**

States: `IDLE -> EVALUATING -> ENTERING -> POSITIONED -> SETTLING -> IDLE`

- `IDLE`: Waiting for next 15m window.
- `EVALUATING`: Reading orderbook, computing edge, checking pre-conditions.
- `ENTERING`: Placing limit order. Timeout = 30s. If not filled, cancel and return to IDLE.
- `POSITIONED`: Monitoring position. Log mid-market every 60s.
- `SETTLING`: Window ended, awaiting settlement confirmation from Kalshi API.
- Back to `IDLE` after settlement logged.

Persistent state in Supabase. On restart: fetch open positions from Kalshi API, reconcile with local state, resume correct FSM state.

---

## 8. Backtesting Requirements (Before Live)

- [ ] Minimum 30 days of historical 15m BTC data.
- [ ] Simulated over/under outcomes for each 15m window.
- [ ] Fee-adjusted PnL computed for every simulated trade.
- [ ] Sharpe ratio > 1.5 required for live approval.
- [ ] Maximum drawdown < 15% of starting capital.
- [ ] Win rate > 55% (note: on a binary over/under, 50% is random; edge shows above ~53%).

---

## 9. Deployment Path

1. **Paper mode** (current): All logic runs, orders are simulated, PnL is logged.
2. **Safe-live mode**: Real orders, $1 max per trade, 5 trades/day max.
3. **Live mode**: Full position sizing, approved after 200+ paper trades with positive edge.

---

## 10. Related Documents

- `research/mystic/mystic-performance-summary.md` - Mystic Bot edge analysis.
- `research/adapted/kalshi-15m-bot-15m.md` - Donor codebase analysis.
- `docs/pdfs/zerotrading-core-execution-statemachine.pdf` - FSM spec.
- `docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf` - Accounting spec.
- `docs/SYSTEM-SYNTHESIS.md` - Cross-system synthesis.
- `ai/handoffs/CURRENT-STATE.md` - Live pointer.
- `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md` - **REQUIRED READING for all agents.**
