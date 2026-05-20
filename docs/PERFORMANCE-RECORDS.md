# Performance Records

> **Purpose:** Auditable log of all performance claims, benchmarks, and live results referenced by ZeroTrading.
> **Policy:** Every entry must include: source, date range, raw data, caveats, and verification status.

---

## Record Format

Each record follows this structure:

```
### [RECORD-ID] - [Short title]
- **Source:** [Where the data came from]
- **Date range:** [Start - End]
- **Verification:** [Verified / Unverified / Partially verified]
- **Data:** [Table or summary]
- **Caveats:** [Known limitations]
```

---

## Records

### PERF-001 - Mystic Bot KXBTC15M Live Results (April 2026)

- **Source:** Public Kalshi community posts with timestamped screenshots of real fills.
- **Date range:** 2026-04-12 to 2026-04-15 (4 trading days).
- **Verification:** Partially verified. Screenshots show real Kalshi fills. No independent audit of full trade log. No API export provided.

#### Summary Data

| Metric | Value |
|---|---|
| Total trades | 137 |
| Wins | 134 |
| Losses | 3 |
| Win rate | 97.8% |
| Estimated gross PnL | $600-800 |
| Estimated net PnL (after fees) | Unknown - fee breakdown not disclosed |
| Avg position size | ~$3-5 per contract |
| Strategy type | Convergence / stay-in-range (inferred) |
| Automation | Fully automated (claimed) |

#### Caveats

1. **Sample size:** 137 trades over 4 days is statistically insufficient to validate edge persistence. Minimum 500+ trades recommended.
2. **Selection bias:** Author may have begun posting only after an initial winning streak. We have no data on prior losing periods.
3. **Fee-adjusted returns unknown:** Kalshi charges 7% winner fee + exchange fees. On $3-5 contracts, this can consume 30-50% of gross edge.
4. **Volatility regime:** April 12-15, 2026 was a relatively calm BTC period. Performance during high-vol regimes is unknown.
5. **Scalability:** $3-5 positions. At larger sizes, fill rates and slippage may degrade the edge.
6. **No drawdown data:** Maximum intraday loss, maximum streak loss, and worst-case scenarios are not reported.
7. **Architecture unknown:** No public code, no disclosed infrastructure. Cannot verify automation claims.

#### Assessment

**Credible as a real-money proof-of-concept.** The data is consistent with a convergence strategy on calm-market BTC 15m contracts. It confirms that the edge exists in short bursts but does NOT prove durability.

**ZeroTrading action:** Use as motivation and directional guidance. Do NOT use as a backtest or as evidence of strategy viability. Independent backtesting required before any live deployment.

---

### PERF-002 - ZeroTrading Paper Mode (Pending)

- **Source:** ZeroTrading bot, Supabase `trades` table.
- **Date range:** TBD (not yet deployed).
- **Verification:** Will be fully verified via Supabase logs + Kalshi API reconciliation.
- **Data:** TBD.
- **Caveats:** Paper mode does not account for real fill rates, slippage, or liquidity constraints.

---

## Verification Policy

1. **Self-reported data** (e.g., community posts, screenshots) is always marked "Partially verified" or "Unverified."
2. **ZeroTrading's own data** must be reconciled against Kalshi API fill records. Discrepancies logged in `ai/summaries/DECISION-LOG.md`.
3. **No performance claim is used for go-live decisions** unless independently verified against exchange records with fee-adjusted PnL.
4. **Minimum sample sizes for strategy approval:**
   - Paper mode: 200+ trades.
   - Safe-live mode: 500+ trades (combined paper + safe-live).
   - Full live mode: 1000+ trades with Sharpe > 1.5 and max drawdown < 15%.
