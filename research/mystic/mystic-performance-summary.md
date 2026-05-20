# Mystic Bot (KXBTC15M) — Performance Summary & Edge Analysis

> **Status:** Research reference — NOT a proven strategy yet for ZeroTrading.
> **Source:** Public Kalshi community posts, live trading screenshots, April 2026.
> **Author of analysis:** Comet on behalf of meszaroszack.

---

## 1. What is Mystic?

Mystic is a community-built automated trading bot targeting Kalshi's KXBTC15M (Bitcoin 15-minute price movement) contracts. It gained attention in April 2026 for posting live-money results with unusually high win rates.

### Key claims (April 12–15, 2026)

| Metric | Claimed Value | Caveat |
|---|---|---|
| Win / Loss | 134 W / 3 L | 4-day window only; survivorship bias possible |
| Win rate | ~97.8% | Short sample; does not account for edge decay |
| Gross PnL | ~$600–800 (est.) | Exact figures vary by post; fee-adjusted PnL unclear |
| Avg position size | $3–5 per contract | Small stakes; scalability unproven |
| Trading window | Continuous 15m intervals | Likely skips low-liquidity hours |
| Automation | Fully automated | Claimed; no public code or architecture disclosed |

### Why these claims matter

- This is the **most transparent public performance data** we've found for any KXBTC15M bot.
- The author posted timestamped screenshots showing actual Kalshi fills, not simulated results.
- The 97.8% win rate on a binary contract suggests a **convergence/mean-reversion edge** — betting that BTC stays within a range during 15m windows.

---

## 2. Inferred Edge Logic

Based on the win/loss pattern and contract structure, the likely strategy is:

### Convergence Wedge ("Launch Wedge")

1. **Observe** BTC spot price at the start of a 15-minute window.
2. **Calculate** the probability that BTC stays within the contract's strike range (e.g., ±$250).
3. **If** the implied probability from the Kalshi orderbook is significantly lower than the model's estimate → **buy YES** (betting price stays in range).
4. **If** the spread is too thin or volatility is spiking → **skip** the window.

### Why this works (when it works)

- BTC 15-minute moves are normally distributed with fat tails. ~85–92% of 15m candles stay within ±$250.
- Kalshi contract pricing often underestimates this "stay in range" probability during calm periods.
- The edge is the gap between the market's implied probability and the realized frequency.

### Why it can fail

- **Tail events:** A single BTC flash crash can wipe 30+ wins.
- **Liquidity drying up:** Wide spreads eliminate the edge.
- **Regime change:** If BTC enters a high-vol regime (news, FOMC, etc.), the base rate drops from ~90% to ~60%.
- **Fee drag:** Kalshi's 7% winner fee + exchange fees erode thin edges quickly.

---

## 3. Credibility Assessment

### What's credible

- Timestamped screenshots of real Kalshi fills.
- Consistent with known BTC 15m volatility distribution.
- The strategy logic (convergence betting) is sound in calm markets.

### What's NOT proven

- No public code or architecture to verify automation claims.
- 4-day sample (April 12–15) is statistically insufficient for edge validation.
- No drawdown data — we don't know the worst single-day loss.
- No fee-adjusted PnL breakdown (Kalshi fees can eat 30–50% of gross edge on cheap contracts).
- Selection bias: author may have started posting only after a winning streak.

### Verdict

**Credible as a real-money demonstration, insufficient as strategy validation.**
The Mystic data confirms that convergence-style KXBTC15M strategies can generate short-term profits. It does NOT prove long-term viability, risk-adjusted returns, or scalability.

---

## 4. What ZeroTrading Should Learn From Mystic

### Adopt

1. **Convergence-first strategy:** Start with "BTC stays in range" as the default hypothesis.
2. **Window selection:** Skip high-volatility windows (news, low liquidity, pre/post market opens).
3. **Small position sizing:** Keep risk per trade tiny until edge is statistically validated (500+ trades minimum).
4. **Transparent performance logging:** Every fill, every fee, every skip — logged to Supabase.

### Reject / Improve

1. **No persistent state:** Mystic's architecture is unknown, but ZeroTrading MUST have Supabase-backed state.
2. **No fee accounting:** ZeroTrading must track gross PnL, net PnL (after fees), and fee drag per strategy.
3. **No risk guardrails documented:** ZeroTrading must implement daily loss limits, streak-based circuit breakers, and volatility regime detection.
4. **No backtesting evidence:** Before live deployment, ZeroTrading must backtest convergence logic against historical 15m BTC data (minimum 30 days).

---

## 5. Open Questions for Further Research

- [ ] Can we obtain Mystic's actual entry/exit price data (not just W/L counts)?
- [ ] What is the fee-adjusted edge per trade? (Need: avg buy price, avg sell/settlement price, Kalshi fee schedule.)
- [ ] How does the win rate degrade during high-vol regimes (e.g., CPI release days)?
- [ ] What is the optimal strike distance for convergence bets? (±$150? ±$250? ±$500?)
- [ ] Does time-of-day matter? (Asian session vs US session vs overnight?)

---

## 6. Related Files

- `research/adapted/kalshi-15m-bot-15m.md` — Prior KXBTC15M bot analysis (modular Python architecture).
- `research/adapted/trader-retro-15m.md` — Prior TS bot analysis (signals on wrong instrument).
- `docs/pdfs/zerotrading-core-execution-statemachine.pdf` — Execution FSM (do not regress).
- `docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf` — Fee accounting (do not regress).
- `docs/SYSTEM-SYNTHESIS.md` — Cross-system synthesis.
