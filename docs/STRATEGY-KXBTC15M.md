# STRATEGY: KXBTC15M Convergence v2

> **Version:** 2.0 (Mystic-informed)
> **Status:** PAPER MODE — not approved for live trading.
> **Last updated:** 2026-05-20
> **Authoritative references:** See `docs/pdfs/` for execution FSM and accounting.

---

## 1. Strategy Overview

**Thesis:** During low-to-moderate volatility regimes, BTC 15-minute price moves stay within predictable ranges ~85–92% of the time. When Kalshi contract pricing underestimates this probability, there is a positive-EV opportunity to buy YES on "price stays in range" contracts.

**Edge source:** Gap between realized stay-in-range frequency and market-implied probability.

**Inspired by:** Mystic Bot live-money results (April 2026). See `research/mystic/mystic-performance-summary.md`.

---

## 2. Entry Rules

### Pre-conditions (ALL must be true)

1. **Volatility regime = CALM or MODERATE.** (Defined below.)
2. **Spread <= $0.04** on the target contract.
3. **Time filter:** Skip first 5 minutes and last 2 minutes of each 15m window.
4. **No scheduled macro events** within the current or next window (FOMC, CPI, NFP, etc.).
5. **Daily loss limit NOT hit.**

### Entry signal

- Identify the KXBTC15M contract whose strike range brackets the current BTC spot price.
- Read the YES ask price from the Kalshi orderbook.
- Compute model probability: `P_model = historical_stay_rate(current_vol_regime, strike_distance)`.
- Compute implied probability: `P_implied = YES_ask_price / 100`.
- **Enter if:** `P_model - P_implied >= EDGE_THRESHOLD` (default: 0.08, i.e., 8% edge).

### Position sizing

- **Max contracts per trade:** 10 (paper mode) / TBD (live).
- **Max risk per trade:** $50 (paper) / TBD (live).
- **Max concurrent positions:** 1 (single-contract focus initially).

---

## 3. Exit Rules

- **Primary exit:** Hold to settlement. (Contract resolves at end of 15m window.)
- **Early exit (optional):** If BTC moves >60% of strike distance before T-5min, market-sell YES to cut losses.
- **No averaging down.** If the position is losing, do not add.

---

## 4. Volatility Regime Classification

| Regime | Definition | Action |
|---|---|---|
| CALM | 1h realized vol < 0.3% | Trade normally |
| MODERATE | 1h realized vol 0.3-0.6% | Trade with tighter edge threshold (+2%) |
| HIGH | 1h realized vol 0.6-1.0% | Reduce position size by 50% |
| EXTREME | 1h realized vol > 1.0% OR scheduled macro event | **NO TRADE** |

---

## 5. Risk Guardrails

| Guardrail | Threshold | Action |
|---|---|---|
| Daily loss limit | -$100 (paper) | Halt all trading for remainder of day |
| Consecutive loss streak | 3 losses in a row | Pause 1 hour, then reassess vol regime |
| Weekly loss limit | -$300 (paper) | Halt trading, trigger manual review |
| Max daily trades | 40 | Hard stop, no more entries |
| Spread blowout | Spread > $0.06 during position | Log warning, do not enter new trades |

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
- [ ] Simulated Kalshi orderbook (or historical snapshots if available).
- [ ] Fee-adjusted PnL computed for every simulated trade.
- [ ] Sharpe ratio > 1.5 required for live approval.
- [ ] Maximum drawdown < 15% of starting capital.
- [ ] Win rate > 80% across all vol regimes combined.

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
