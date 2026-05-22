# CURRENT STATE

_Live pointer file. Overwrite at end of every session. Append-only history lives in `ai/summaries/DECISION-LOG.md`._

**Last updated:** 2026-05-21 21:33 ET  
**Updated by:** Perplexity Computer (Comet) on behalf of meszaroszack  
**Reason for update:** End of session — ZeroTrading Core stabilization (11 bug fixes) + governance integration into zerotrading-core

---

## CRITICAL: Know your market before doing ANYTHING

**`ai/guardrails/KALSHI-MARKET-REFERENCE.md`** is REQUIRED READING.

This repo targets TWO different Kalshi BTC markets. They have DIFFERENT structures:

| System | Market | Structure | Status |
|---|---|---|---|
| ZeroTrading Core | **KXBTCD** (hourly) | Ladder of strike prices; above/below threshold | **LIVE** on Railway |
| ZeroTrading 15M | **KXBTC15M** (15-min) | Single strike = current price; simple up/down | **Phase 1 skeleton complete** |

**Do NOT mix strategies between markets.**

---

## ZeroTrading Core (KXBTCD - Hourly) Status

- **Deployed at:** `zerotrading-core-production.up.railway.app`
- **Mode:** LIVE (real money)
- **Last known balance:** ~$26.42 (as of 2026-05-21 ~21:06 ET)
- **Cycles this session:** 3+ completed (2 wins, 1 loss per operator)
- **Latest commit:** `f4b7bdc` — "fix: fill price field names, re-entry guard, exit P&L reporting"
- **Build status:** Passing (TypeScript clean)
- **Governance:** `ai/` scaffold, `.cursor/rules/`, `CHANGELOG.md`, `CONTRIBUTING.md`, `SECURITY.md`, `README.md` now all present in zerotrading-core

### Do Not Regress (Core)
- Settlement: `GET /markets/{ticker}` → `status==="finalized"` — positions endpoint dropped `settled` field Dec 2025
- Fill prices: `no_price_dollars` / `yes_price_dollars` — not `price_dollars`
- `position_fp`: Kalshi decimal string × 10000 normalized at adapter boundary
- Time-exit intentionally removed — do not re-add
- Re-entry guard: 60-min cooldown per ticker (`_lastCompletedTicker`)
- `positionStore.create()` via `ACCT_OPEN_POSITION` effect — not inline in FSM

### Open Items (Core)
- Orphaned Kalshi positions from pre-fix-6 bugs require manual review on Kalshi dashboard
- No automated test harness exists — Phase 2 item
- Exit P&L fields not yet verified in live cycle (fix #11 — verify in next cycle)

---

## Beta Repo Status (ZeroTrading 15M)

**[meszaroszack/zerotradingx15minbtc](https://github.com/meszaroszack/zerotradingx15minbtc)**

| Phase | Status | Commit |
|-------|--------|--------|
| Phase 0 — Documentation scaffold | ✅ Complete | `74192c7` |
| Phase 1 — Python skeleton | ✅ Complete | `d205ec4` |
| Phase 2 — First paper run | ⏳ Not started | — |

**Phase 1 delivered (2026-05-21 21:00 ET):**
- 20 source files covering all 9 architectural layers
- Kalshi REST + WS clients (async, RSA auth)
- Binance 1m kline stream, feed health monitor
- Supabase-backed FSM (5 states, duplicate-trade guard)
- Black-Scholes terminal probability model, 7 skip gates
- Paper mode with full simulation
- Railway deploy config

### Before First Paper Run (Manual Steps Required)
1. Apply `src/db/schema.sql` to Supabase
2. Set Railway env vars (all listed in beta `.env.example`)
3. Set `KALSHI_MARKET_TICKER` to next active KXBTC15M market
4. Deploy to Railway
5. Validate `decisions` and `positions` tables receive records on first loop

---

## Next Steps (in order)

1. **Watch next Core live cycle** — verify exit P&L fields populate in dashboard after fix #11
2. **Manually resolve orphaned Core positions** on Kalshi exchange
3. **Apply schema + deploy 15M beta** — first paper run
4. **Monitor 15M paper run** — check `decisions` table for skip reasons
5. **Phase 2 (15M):** live order execution, macro gate, auto-ticker discovery, consecutive-loss cooldown

---

## Research Inventory

All prior build research is committed in `research/adapted/`. Key files:

| File | What it contains |
|------|-----------------|
| `kalshi-public-docs-api-websocket.md` | Authoritative Kalshi API/WS reference |
| `kalshi-btcd-trader-hourly.md` | BS probability model, circuit breaker patterns |
| `alpha-bot-15m.md` | Supabase persistence patterns |
| `probablyprofit-order-management.md` | OrderManager patterns |
| `morningside-wagewise-orderbook.md` | L3 orderbook architecture |
| `phase1-implementation-patterns.md` | Phase 1 design decisions (15M) |

Guardrails in `ai/guardrails/`:
- `PERSISTENCE-RECONCILIATION.md` — 6 rules, 10-question self-test (REQUIRED)
- `KXBTC15M-MARKET-STRUCTURE.md` — 15M market structure rules (REQUIRED for 15M work)
- `KALSHI-MARKET-REFERENCE.md` — API field mappings and fee schedule (REQUIRED)
