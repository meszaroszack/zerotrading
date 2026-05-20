# Mystic Bot (KXBTC15M) - Performance Summary & Edge Analysis

> **Status:** Research reference - NOT a proven strategy yet for ZeroTrading.
> **Source:** Public Kalshi community posts, live trading screenshots, April 2026.
> **Author of analysis:** Comet on behalf of meszaroszack.
> **v2 - CORRECTED:** Prior version incorrectly described KXBTC15M as a range/wedge product. It is a simple binary over/under. See `docs/STRATEGY-KXBTC15M.md` section "CRITICAL: KXBTC15M Market Structure."

---

## IMPORTANT: KXBTC15M is Over/Under, Not Range/Wedge

**Every AI agent must read this before interpreting Mystic's results.**

KXBTC15M is a binary contract: "Will BTC be ABOVE or BELOW the strike price at the end of this 15-minute window?" You buy YES (over) or NO (under). There are no ranges, no wedges, no bands, no convergence zones. One strike, two outcomes.

Any prior analysis in this repo that referenced ranges, wedges, or "stay in range" mechanics was WRONG and has been corrected.

---

## 1. What is Mystic?

Mystic is a community-built automated trading bot targeting Kalshi's KXBTC15M contracts. It gained attention in April 2026 for posting live-money results with unusually high win rates.

### Key claims (April 12-15, 2026)

| Metric | Claimed Value | Caveat |
|---|---|---|
| Win / Loss | 134 W / 3 L | 4-day window only; survivorship bias possible |
| Win rate | ~97.8% | Short sample; does not account for edge decay |
| Gross PnL | ~$600-800 (est.) | Exact figures vary by post; fee-adjusted PnL unclear |
| Avg position size | $3-5 per contract | Small stakes; scalability unproven |
| Trading window | Continuous 15m intervals | Likely skips low-liquidity hours |
| Automation | Fully automated | Claimed; no public code or architecture disclosed |

### Why these claims matter

- This is the **most transparent public performance data** we've found for any KXBTC15M bot.
- The author posted timestamped screenshots showing actual Kalshi fills, not simulated results.
- A 97.8% win rate on a binary over/under contract implies the bot has a **strong directional prediction model** - it is correctly calling whether BTC will finish above or below the strike ~98% of the time.

---

## 2. Inferred Edge Logic

Based on the win/loss pattern and the over/under contract structure, the likely strategy is:

### Directional Bias Detection

1. **Observe** BTC price action in the moments after a 15m window opens (the strike is set).
2. **Detect** short-term directional bias using momentum, order flow, or microstructure signals.
3. **If** the model predicts BTC will finish above strike with high confidence AND the YES price is below model's fair value -> **buy YES (over)**.
4. **If** the model predicts BTC will finish below strike with high confidence AND the NO price is below model's fair value -> **buy NO (under)**.
5. **If** no clear directional signal -> **skip** the window.

### Why a ~98% win rate is possible on over/under

- The strike is set at the start of the window. If BTC has even a tiny directional drift in the first few minutes, the probability of it finishing on that side increases rapidly as time passes.
- **Key insight:** Mystic likely waits until there's a clear short-term trend within the window, then buys the side that's already winning at a price that still offers value. As the window progresses and BTC stays on one side of the strike, that side's price approaches $1.00.
- This is essentially a **momentum/trend-following strategy within each 15m window**, not a prediction made at the start.

### Why it can fail

- **Reversals:** BTC moves one direction early, Mystic buys that side, then BTC reverses in the last few minutes.
- **Chop/noise:** In sideways markets where BTC oscillates around the strike, directional signals are unreliable.
- **Liquidity drying up:** Wide spreads eliminate the edge.
- **Fee drag:** Kalshi's 7% winner fee + exchange fees erode thin edges on cheap contracts.
- **Latency:** If the strategy depends on reacting to intra-window momentum, execution speed matters.

---

## 3. Credibility Assessment

### What's credible

- Timestamped screenshots of real Kalshi fills.
- The over/under structure makes high win rates plausible IF the bot is selective about when to enter (skip ambiguous windows, only trade when momentum is clear).
- Consistent with known BTC microstructure: short-term momentum is real and exploitable.

### What's NOT proven

- No public code or architecture to verify automation claims.
- 4-day sample (April 12-15) is statistically insufficient for edge validation.
- No drawdown data - we don't know the worst single-day loss.
- No fee-adjusted PnL breakdown (Kalshi fees can eat 30-50% of gross edge on cheap contracts).
- Selection bias: author may have started posting only after a winning streak.
- **We don't know WHEN in the 15m window Mystic enters.** Early entry (more risk, more reward) vs late entry (less risk, less reward) changes the entire risk profile.

### Verdict

**Credible as a real-money demonstration, insufficient as strategy validation.**
The Mystic data confirms that directional momentum on KXBTC15M over/under contracts can generate short-term profits. It does NOT prove long-term viability, risk-adjusted returns, or scalability.

---

## 4. What ZeroTrading Should Learn From Mystic

### Adopt

1. **Directional momentum as primary signal:** The edge is in predicting which side of the strike BTC finishes, likely using intra-window momentum.
2. **Window selection:** Skip ambiguous/choppy windows. Only trade when there's a clear directional signal.
3. **Small position sizing:** Keep risk per trade tiny until edge is statistically validated (500+ trades minimum).
4. **Transparent performance logging:** Every fill, every fee, every skip - logged to Supabase.

### Reject / Improve

1. **No persistent state:** Mystic's architecture is unknown, but ZeroTrading MUST have Supabase-backed state.
2. **No fee accounting:** ZeroTrading must track gross PnL, net PnL (after fees), and fee drag per strategy.
3. **No risk guardrails documented:** ZeroTrading must implement daily loss limits, streak-based circuit breakers, and volatility regime detection.
4. **No backtesting evidence:** Before live deployment, ZeroTrading must backtest directional logic against historical 15m BTC data (minimum 30 days).

---

## 5. Open Questions for Further Research

- [ ] Can we obtain Mystic's actual entry/exit price data (not just W/L counts)?
- [ ] At what point in the 15m window does Mystic enter? (This determines the risk/reward profile.)
- [ ] What is the fee-adjusted edge per trade?
- [ ] How does the win rate degrade during high-vol or choppy regimes?
- [ ] What directional signals does Mystic use? (Momentum? Order flow? Both?)
- [ ] Does time-of-day matter? (Asian session vs US session vs overnight?)

---

## 6. Related Files

- `docs/STRATEGY-KXBTC15M.md` - Strategy spec (v3, corrected to over/under).
- `research/adapted/kalshi-15m-bot-15m.md` - Prior KXBTC15M bot analysis (modular Python architecture).
- `research/adapted/trader-retro-15m.md` - Prior TS bot analysis (signals on wrong instrument).
- `docs/pdfs/zerotrading-core-execution-statemachine.pdf` - Execution FSM (do not regress).
- `docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf` - Fee accounting (do not regress).
- `docs/SYSTEM-SYNTHESIS.md` - Cross-system synthesis.
- `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md` - **REQUIRED READING for all agents.**
