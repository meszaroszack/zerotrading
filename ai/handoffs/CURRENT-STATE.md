# CURRENT STATE

_Live pointer file. Overwrite at end of every session. Append-only history lives in `ai/summaries/DECISION-LOG.md`._

**Last updated:** 2026-05-23 13:50 ET
**Updated by:** Perplexity Computer (Comet) on behalf of meszaroszack
**Reason for update:** FIX-CORE-010 merged across two PRs (#4 `f809f18`, #5 `6e495cf`). The bot's exit logic was silently disabled whenever the Kalshi orderbook REST endpoint returned null — same gate that blanked the dashboard right-column also gated TP/SL. Reference patterns from `kalshi-btcd-trader` and `kalshi-15m-bot` confirmed the fix. Operator cleared Kalshi positions and bot is now IDLE on deployed `6e495cf` with balance $28.13. See CRASH-AND-FIX-LOG FIX-CORE-010.

---

## CRITICAL: Required reading for every new agent / new session

Read in this order before touching any code:

1. `AGENTS.md` (repo root) — seven hard rules, repository map, end-of-session checklist.
2. `ai/checklists/SESSION-STARTUP-CHECKLIST.md` — 60-second shell block + four pre-flight questions.
3. `ai/guardrails/BRANCH-AND-MERGE-DISCIPLINE.md` — branch/merge contract and worked-example incident from 2026-05-23.
4. `ai/guardrails/PERSISTENCE-RECONCILIATION.md` — persistence contract.
5. `ai/guardrails/KALSHI-MARKET-REFERENCE.md` — market structure (KXBTCD vs KXBTC15M).
6. This file.
7. `ai/summaries/CRASH-AND-FIX-LOG.md` — every bug ever shipped, with hard rules.

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
- **Latest commit on main:** `6e495cf` — "fix(strategy): TP/SL fallback when orderbook REST returns null (#5)" (FIX-CORE-010 PR 2/2). Prior: `f809f18` (#4) display-side fix.
- **Build status:** server typecheck clean, **client typecheck clean** (manually verified), **82/82 tests passing** across 6 files (`endOfHour.test.ts` 13→20, new `derivePositionUiFields.test.ts` 7), lint 0 errors / 11 pre-existing warnings, `vite build` clean.
- **Governance:** `ai/` scaffold, `.cursor/rules/`, `CHANGELOG.md`, `CONTRIBUTING.md`, `SECURITY.md`, `README.md`, eslint scaffold (`.eslintrc.cjs`, `.eslintignore`, `tsconfig.eslint.json`)
- **Railway service config (set via dashboard):** `STATE_DIR=/data`, volume `zt-core-state` mounted at `/data`, source branch = `main`

### Do Not Regress (Core)
- Settlement: `GET /markets/{ticker}` → `status==="finalized"` — positions endpoint dropped `settled` field Dec 2025
- Fill prices: `no_price_dollars` / `yes_price_dollars` — not `price_dollars`
- `position_fp`: Kalshi decimal string × 10000 normalized at adapter boundary
- **Time-exit intentionally removed — do not re-add** (main removed it in `be99ad2`)
- Re-entry guard: 60-min cooldown per ticker (`_lastCompletedTicker`)
- `positionStore.create()` via `ACCT_OPEN_POSITION` effect — not inline in FSM
- tradeHistory: PositionRecord-outcome-only mapping (do not re-introduce ledger-join — main owns this contract)
- bidSeries: persisted ring buffer at `STATE_DIR/bid-series.json`, capacity 600, sampled only in ENTERING/OPEN/EXITING
- **FIX-CORE-010: Position UI fields must read from multiple sources — not just `snap.exitPosture`.** Use the exported `derivePositionUiFields()` helper in `src/server/index.ts`. Do not refactor it back to a single-source read.
- **FIX-CORE-010: `handleOpen()` must fetch market summaries BEFORE the orderbook gate** and synthesize a fallback book via `synthesizeOrderbookFromSummary()` when REST returns null. The market-finalized reconcile branch fires only when BOTH sources are unusable. Do not re-introduce the old single-gate early-return.

### Open Items (Core)
- **TECH DEBT — Root `npm run typecheck` only invokes server `tsc`.** The client lives under `client/tsconfig.json` and was never wired in. Why FIX-CORE-009 reached production. Next PR: change root `typecheck` script to run `tsc --noEmit && tsc --noEmit -p client/tsconfig.json`. (Still open after FIX-CORE-010.)
- **TECH DEBT — No UI smoke test.** Add a minimal headless-browser test that loads the served bundle and asserts `document.getElementById('root').children.length > 0` and console has no errors. Would have caught FIX-CORE-009 immediately. (Still open after FIX-CORE-010.)
- **TECH DEBT — No integration test for `handleOpen()` exit path.** PR #5 added unit tests on `synthesizeOrderbookFromSummary()` but `handleOpen` itself remains untested. Adding a test harness that drives `handleOpen` through ENTERING/OPEN with mocked adapters would have caught FIX-CORE-010 years ago. Track when prioritized.
- **TECH DEBT — Audit-log signal `orderbookFromFallback: true` is unmetered.** PR #5 added this flag to `URGENT_EXIT_TICK` events but nothing alerts on a non-zero rate. If REST instability becomes chronic the bot will silently run on summary-only data — functional, but operator should know. Wire to a counter and a /api/status surface.
- **OPERATOR ACTION REQUIRED — Branch protection on `meszaroszack/zerotrading-core`:** require PR review, require linear history (rebase or squash), require status checks green, require branch up-to-date with base before merge. Without this, the 2026-05-23 branch-divergence incident can recur.
- **OPERATOR ACTION REQUIRED — Same branch protection on `meszaroszack/zerotrading`** (parent).
- **OPERATOR ACTION REQUIRED — Delete stale branches** on `zerotrading-core`: `core-engine-review-v1`, `fix/core-stabilization-v1`, `reconcile/core-stabilization-v1`.
- Orphaned Kalshi positions from pre-fix-6 bugs require manual review on Kalshi dashboard.
- Lint warnings (11) are all pre-existing unused-var noise — schedule a cleanup PR but they are not blocking.

---

## Beta Repo Status (ZeroTrading 15M)

**[meszaroszack/zerotradingx15minbtc](https://github.com/meszaroszack/zerotradingx15minbtc)**

| Phase | Status | Commit |
|-------|--------|--------|
| Phase 0 — Documentation scaffold | Complete | `74192c7` |
| Phase 1 — Python skeleton | Complete | `d205ec4` |
| Phase 2 — First paper run | Not started | — |

### Before First Paper Run (Manual Steps Required)
1. Apply `src/db/schema.sql` to Supabase
2. Set Railway env vars (all listed in beta `.env.example`)
3. Set `KALSHI_MARKET_TICKER` to next active KXBTC15M market
4. Deploy to Railway
5. Validate `decisions` and `positions` tables receive records on first loop

---

## Next Steps (in order)

1. **Operator: enable branch protection** on both `zerotrading-core` and `zerotrading` `main` branches.
2. **Operator: delete stale branches** on `zerotrading-core`.
3. **Watch next Core live cycle** — verify bidSeries sparkline renders and `health.json` updates.
4. **Manually resolve orphaned Core positions** on Kalshi exchange.
5. **Apply schema + deploy 15M beta** — first paper run.
6. **Multi-repo governance pass** — operator has multiple repos (some production, some research-only). Need to decide which get full `ai/` scaffold and which get a lighter `AGENTS.md`-only instruction layer.

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
- `BRANCH-AND-MERGE-DISCIPLINE.md` — branch/merge contract (REQUIRED, new 2026-05-23)
- `PERSISTENCE-RECONCILIATION.md` — 6 rules, 10-question self-test (REQUIRED)
- `KXBTC15M-MARKET-STRUCTURE.md` — 15M market structure rules (REQUIRED for 15M work)
- `KALSHI-MARKET-REFERENCE.md` — API field mappings and fee schedule (REQUIRED)
