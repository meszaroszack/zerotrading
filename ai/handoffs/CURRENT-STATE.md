# CURRENT STATE

_Live pointer file. Overwrite at end of every session. Append-only history lives in `ai/summaries/DECISION-LOG.md`._

**Last updated:** 2026-05-21 21:32 ET  
**Updated by:** Perplexity Computer (Comet) on behalf of meszaroszack  
**Reason for update:** Phase 1 paper mode bug fixes (5 crash-level bugs) + auto-discovery — beta commit `914351a`

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

## Beta Repo Status

**[meszaroszack/zerotradingx15minbtc](https://github.com/meszaroszack/zerotradingx15minbtc)**

| Phase | Status | Commit |
|-------|--------|--------|
| Phase 0 — Documentation scaffold | ✅ Complete | `74192c7` |
| Phase 1 — Python skeleton | ✅ Complete | `d205ec4` |
| Phase 1 — Paper mode bug fixes | ✅ Complete | `914351a` |
| Phase 2 — First paper run | ⏳ Not started | — |

**Phase 1 delivered (2026-05-21 21:00 ET, commit `d205ec4`):**
- 20 source files covering all 9 architectural layers
- Kalshi REST + WS clients (async, RSA auth, all 3 WS channels)
- Binance 1m kline stream
- Feed health monitor
- Boot + periodic reconciliation
- Supabase-backed FSM (5 states, duplicate-trade guard)
- Paper mode (full simulation, real Supabase records, guardrails)
- Black-Scholes terminal probability model, 7 skip gates
- Fee formula, P&L booking, daily summary
- Structured JSON logger
- main.py (boot → reconcile → paper loop)
- Railway deploy config (Procfile, railway.toml, requirements.txt)
- Supabase schema (6 tables, aligned to Python column names)

**Phase 1 bug fixes (2026-05-21 21:32 ET, commit `914351a`):**
- Bug 1 fixed: `get_supabase()` ImportError on boot — `supabase.py` rewritten with `get_supabase()` + `db_run()` async wrapper
- Bug 2 fixed: `Position.entry_fill_cost_dollars` added to dataclass + `from_row()`
- Bug 3 fixed: `decisions` table dedup — `_last_logged_window` prevents 60 NO_TRADE rows per window
- Bug 4 fixed: paper mode uses `_simulated_balance` (starts $200); safety floor no longer queries real Kalshi demo account ($0)
- Bug 5 fixed: all supabase-py calls wrapped in `asyncio.to_thread()` via `db_run()` — permanent architectural rule
- Auto-discovery added: `discover_active_kxbtc15m()` queries Kalshi REST; `KALSHI_MARKET_TICKER` is now optional; ticker re-discovered on every IDLE iteration for automatic window rotation

**Parent repo updated in Phase 1 cross-repo sync session:**
- `docs/ARCHITECTURE-KXBTC15M.md` — NEW (full implementation architecture)
- `research/adapted/phase1-implementation-patterns.md` — NEW (10 design decisions)
- `ai/summaries/DECISION-LOG.md` — APPENDED (9 architectural decisions)
- `CHANGELOG.md` — APPENDED
- `ai/summaries/2026-05-21-2100-phase1-cross-repo-sync.md` — NEW (session summary)

**Parent repo updated in bug-fix session (this update):**
- `docs/ARCHITECTURE-KXBTC15M.md` — UPDATED (db_run pattern, simulated balance, auto-discovery, boot sequence, env vars, Phase 2 scope)
- `ai/summaries/DECISION-LOG.md` — APPENDED (bug fix decisions)
- `CHANGELOG.md` — APPENDED (bug fix entry)
- `ai/handoffs/CURRENT-STATE.md` — UPDATED (this file)

---

## Before First Paper Run (Manual Steps Required)

1. **Apply `src/db/schema.sql`** to Supabase (Supabase Studio SQL editor)
2. **Set Railway env vars** — required: `KALSHI_API_KEY_ID`, `KALSHI_API_PRIVATE_KEY`, `KALSHI_ENV`, `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `APP_MODE`
3. **`KALSHI_MARKET_TICKER` is now optional** — auto-discovery handles it. Set only if you want to force a specific ticker for testing.
4. **Deploy to Railway** — push to beta main branch triggers build
5. **Validate Supabase tables** — confirm `decisions` and `positions` receive records on first loop

---

## ZeroTrading Core (KXBTCD - Hourly) Status

- Deployed at `zerotrading-core-production.up.railway.app`
- Connected to Kalshi exchange (LIVE, real money)
- Balance: ~$4.16 (as of last check — may have changed)
- Making NO_TRADE decisions (strike too close, net edge too low)
- Architecture docs: `docs/pdfs/` (3 canonical PDFs)

---

## Do Not Regress

- KXBTCD and KXBTC15M are DIFFERENT markets — never mix
- KXBTC15M is a simple binary over/under — ONE strike, TWO outcomes, NO ranges/bands
- Settlement uses 60-second CFB RTI average beginning 1 min BEFORE expiration
- ALL state MUST be persisted in Supabase, NEVER in-memory
- Every boot MUST reconcile with Kalshi before entering the trading loop
- P&L MUST come from actual fills and settlements — never from estimates
- `client_order_id` on every order — idempotency is non-negotiable
- Boot reconciliation raises `ReconcileError` on ORPHANED positions — do not swallow this error

---

## Next Steps (in order)

1. **Apply schema and deploy** — First paper run (see above)
2. **Monitor paper run** — Check `decisions` table for skip reasons; verify sigma is computing
3. **Phase 2 work items:**
   - Live order execution (orders.py stub → real implementation)
   - Macro gate (FOMC/CPI calendar integration)
   - Consecutive-loss cooldown persistence (currently in-memory, resets on restart)

---

## Research Inventory (for next agent or session)

All prior build research is committed in `research/adapted/`. Key files:

| File | What it contains |
|------|-----------------|
| `kalshi-public-docs-api-websocket.md` | Authoritative Kalshi API/WS reference (channels, schemas, rate limits) |
| `kalshi-btcd-trader-hourly.md` | Source of BS probability model, circuit breaker patterns |
| `alpha-bot-15m.md` | Supabase persistence patterns, reference price alignment |
| `probablyprofit-order-management.md` | OrderManager patterns, integrated reconciliation |
| `morningside-wagewise-orderbook.md` | L3 orderbook architecture |
| `phase1-implementation-patterns.md` | **NEW** — Phase 1 design decisions (10 documented choices) |

Guardrails in `ai/guardrails/`:
- `PERSISTENCE-RECONCILIATION.md` — 6 rules, 10-question self-test (REQUIRED before writing trading code)
- `KXBTC15M-MARKET-STRUCTURE.md` — Market structure rules (REQUIRED)
- `KALSHI-MARKET-REFERENCE.md` — API field mappings and fee schedule (REQUIRED)
