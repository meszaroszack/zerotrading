# Crash and Fix Log — ZeroTrading (Parent Platform)

**What this file is:** An append-only log of every crash, bug, runtime failure, or silent data corruption found across any ZeroTrading system — including the x15minBTC beta repo.

**Why it exists:** AI agents write code quickly and confidently. That doesn't mean it's correct. This log exists so that:
- Bugs are never "fixed and forgotten" — they are documented and pattern-matched against future work
- Any new agent reading this file immediately knows what has already broken across the entire platform
- Patterns of failure are visible — if the same root cause hits two different systems, it needs a platform-level guardrail

**Scope:** This file tracks bugs from BOTH repos:
- `meszaroszack/zerotrading` (parent platform, live KXBTCD bot)
- `meszaroszack/zerotradingx15minbtc` (beta, KXBTC15M paper/live bot)

Each entry notes which repo the bug was in. The beta repo has its own `ai/summaries/CRASH-AND-FIX-LOG.md` with the same entries for its bugs.

**Authority:** Append-only. Never delete entries. Supersede by appending a correction entry.

**REQUIRED FOR ALL AI AGENTS:**
> Before writing any code in either ZeroTrading repo, read this file. Before ending any session, check whether a bug was found or fixed and append an entry here AND in the relevant child repo log. This is not optional.

---

## Entry Format

```
### CRASH-NNN — <short title>
**Date:** YYYY-MM-DD HH:MM ET
**Repo:** zerotradingx15minbtc | zerotrading
**Found by:** <human name or agent>
**Severity:** crash | data-loss | silent-wrong | performance | minor
**Status:** fixed | open | mitigated
**Symptom:** What would happen (or did happen) at runtime
**Root cause:** Why it happened
**Fix:** What was changed and in which file(s)
**Commit:** <repo commit hash>
**Pattern:** <pattern tag — see bottom of file>
**Cross-repo:** Was the OTHER repo affected or updated?
```

---

## Crash Log

### CRASH-001 — `get_supabase()` ImportError — bot crashes on boot
**Date:** 2026-05-21 21:32 ET  
**Repo:** zerotradingx15minbtc  
**Found by:** Perplexity Computer (Comet) — paper mode audit, before first paper run  
**Severity:** crash  
**Status:** fixed  
**Symptom:** `ImportError: cannot import name 'get_supabase' from 'src.db.supabase'` on the first import. Bot would never start.  
**Root cause:** `supabase.py` exported a `db` singleton. All other modules imported `get_supabase()` — a function that was never defined. The naming contract between the module and its callers was never enforced.  
**Fix:** Rewrote `supabase.py` to export `get_supabase()` + `db_run()` (see CRASH-005).  
**Commit:** `914351a` (zerotradingx15minbtc)  
**Pattern:** #import-contract-mismatch  
**Cross-repo:** Parent repo `docs/ARCHITECTURE-KXBTC15M.md` updated to document `db_run()` pattern (commit `d9ea610`).

---

### CRASH-002 — `Position.entry_fill_cost_dollars` always `None` at settlement
**Date:** 2026-05-21 21:32 ET  
**Repo:** zerotradingx15minbtc  
**Found by:** Perplexity Computer (Comet) — paper mode audit  
**Severity:** data-loss  
**Status:** fixed  
**Symptom:** `settle()` reads `position.entry_fill_cost_dollars` to compute net P&L. Always `None`. P&L computed incorrectly or crashes on None arithmetic.  
**Root cause:** Field existed in Supabase schema but was never added to the `Position` dataclass or `from_row()`.  
**Fix:** Added `entry_fill_cost_dollars: float` to `Position` dataclass and populated it in `from_row()`.  
**Commit:** `914351a` (zerotradingx15minbtc)  
**Pattern:** #schema-dataclass-drift  
**Cross-repo:** Noted in parent repo DECISION-LOG + CHANGELOG (commit `d9ea610`).

---

### CRASH-003 — Decisions table written 60 times per window (NO_TRADE spam)
**Date:** 2026-05-21 21:32 ET  
**Repo:** zerotradingx15minbtc  
**Found by:** Perplexity Computer (Comet) — paper mode audit  
**Severity:** silent-wrong  
**Status:** fixed  
**Symptom:** After a paper run, `decisions` table contains ~60 identical NO_TRADE rows per 15-minute window. Analytics are polluted.  
**Root cause:** `log_decision()` called on every 5-second loop tick. No per-window dedup.  
**Fix:** Added `_last_logged_window` to `PaperTrader`. NO_TRADE skips if already logged for current window. TRADE always writes.  
**Commit:** `914351a` (zerotradingx15minbtc)  
**Pattern:** #loop-tick-vs-window-event  
**Cross-repo:** Noted in parent repo DECISION-LOG + CHANGELOG.

---

### CRASH-004 — Safety floor permanently blocked paper mode (balance = $0)
**Date:** 2026-05-21 21:32 ET  
**Repo:** zerotradingx15minbtc  
**Found by:** Perplexity Computer (Comet) — paper mode audit  
**Severity:** crash (functional — bot runs but never trades)  
**Status:** fixed  
**Symptom:** Bot evaluates every window, hits safety floor check, returns NO_TRADE forever. Kalshi demo account balance is $0 at account creation. The $50 floor was always breached.  
**Root cause:** Paper mode had no simulated balance concept. `is_allowed_to_trade()` queried the real Kalshi demo account balance.  
**Fix:** Introduced `_simulated_balance` (default $200). Entry costs deducted on fill; payouts credited on settlement. Floor check uses simulated balance in paper mode.  
**Commit:** `914351a` (zerotradingx15minbtc)  
**Pattern:** #paper-live-boundary-confusion  
**Cross-repo:** Parent repo `docs/ARCHITECTURE-KXBTC15M.md` Paper Mode table updated (commit `d9ea610`).

---

### CRASH-005 — `supabase-py` blocks asyncio event loop — WS keepalives stall
**Date:** 2026-05-21 21:32 ET  
**Repo:** zerotradingx15minbtc  
**Found by:** Perplexity Computer (Comet) — paper mode audit  
**Severity:** crash (under sustained operation)  
**Status:** fixed  
**Symptom:** Kalshi and/or Binance WebSocket connections drop with keepalive timeouts during continuous operation.  
**Root cause:** `supabase-py` v2 is synchronous. Called directly from asyncio coroutines, every DB write blocked the entire event loop. WS frames couldn't be processed during DB I/O.  
**Fix:** `db_run(fn)` wrapper uses `asyncio.to_thread()` to offload every supabase-py call to a thread worker. All call sites use `await db_run(lambda db: ...)`.  
**This is a PERMANENT PLATFORM RULE.** Any code in any ZeroTrading repo that uses supabase-py must go through `db_run()`. Direct synchronous calls are forbidden in async contexts.  
**Commit:** `914351a` (zerotradingx15minbtc)  
**Pattern:** #sync-in-async-context  
**Cross-repo:** Parent repo `docs/ARCHITECTURE-KXBTC15M.md` file map updated with permanent db_run note (commit `d9ea610`).

---

## Open Issues

_None currently tracked._

---

## Failure Pattern Index

Use these tags when writing new entries. Add new patterns as new failure modes emerge.

| Tag | Description |
|-----|-------------|
| `#import-contract-mismatch` | Module exports X but callers import Y — naming contract never enforced |
| `#schema-dataclass-drift` | DB schema has a column that the Python dataclass is missing (or vice versa) |
| `#loop-tick-vs-window-event` | Logic that fires once per window was placed inside a per-tick loop |
| `#paper-live-boundary-confusion` | Paper mode code behaves as if it's using real money/accounts |
| `#sync-in-async-context` | Synchronous blocking I/O called from an asyncio coroutine |
| `#state-not-persisted` | State that should survive a restart was kept in memory |
| `#double-booking` | P&L, fees, or records written more than once for the same event |
| `#missing-idempotency-key` | Order/record submitted without a dedup key, enabling duplicates on retry |
| `#reconcile-skipped` | Boot proceeded without reconciliation; ghost/orphaned positions persisted |
| `#hardcoded-env-value` | A value that should come from env was hardcoded in source |
| `#single-source-coupling` | One nullable object is the only path to N downstream fields; when it fails, all N collapse together |
| `#happy-path-only` | Code handles the happy case correctly but lacks a graceful path when an upstream dep returns null/empty/stale |
| `#sibling-pattern-not-propagated` | A correct pattern existed in a sibling repo (or even the same file) but was never propagated to the buggy site |
| `#invisible-trading-failure` | Trading logic silently skipped under a condition that no health probe or alert detects |
| `#partial-checks` | A validation script (typecheck/lint/test) ran but did not cover all subtrees of the repo |
| `#ts-not-enforced-in-jsx-interp` | Vite/React production build does not flag ReferenceError inside JSX template-string interpolations; only `tsc --noEmit` catches them |
| `#ui-only-failure-invisible-to-health-probes` | Server-side health, API, and uptime all pass while the UI is broken |

---

## Cross-Repo Policy

When a bug is found in either repo:

1. **Fix it** in the affected repo first.
2. **Log it here** (parent `CRASH-AND-FIX-LOG.md`) AND in the affected repo's own `CRASH-AND-FIX-LOG.md`.
3. **Ask:** does this bug pattern also exist in the OTHER repo? If yes, fix it there too.
4. **Ask:** does this pattern need a new guardrail? If yes, add it to `ai/guardrails/PERSISTENCE-RECONCILIATION.md` or create a new guardrail doc.
5. **Commit** both repos before ending the session. Do not leave cross-repo log entries uncommitted.

## How to Add an Entry

1. Assign the next sequential ID: `CRASH-NNN`
2. Fill every field — no skipping
3. Tag with at least one pattern from the index
4. Note cross-repo impact explicitly
5. Commit to this repo: `docs(crash-log): CRASH-NNN — <short title>`
6. Commit to the affected child repo with the same message

**If you found a bug mid-session and fixed it, the entry goes here before the session ends. Not "later". Not "in the next session". Now.**

---

### CRASH-006 — `SupabaseException("Invalid API key")` — supabase-py version too old to accept new key format
**Date:** 2026-05-21 22:01 ET  
**Repo:** zerotradingx15minbtc  
**Found by:** Operator (first Railway deploy) + Perplexity Computer (root cause)  
**Severity:** crash  
**Status:** fixed  
**Symptom:** Railway deploy crashes immediately with `raise SupabaseException("Invalid API key")`.  
**Root cause:** `supabase==2.5.0` pinned in `requirements.txt`. Supabase's new `sb_publishable_...` key format requires `supabase-py >= 2.7.0`. The old version rejects the key as invalid regardless of whether the value is correct. Secondary bug: `get_supabase()` had a stale service role key fallback that doesn't belong in the bot at all.  
**Fix:** Bumped to `supabase==2.30.0`. Removed service role key fallback from `get_supabase()`.  
**Commit:** (beta repo — see next commit)  
**Pattern:** #hardcoded-env-value  
**Cross-repo:** Fix in zerotradingx15minbtc. Same entry in beta CRASH-AND-FIX-LOG.md.

**AGENT NOTE:** When you see `Invalid API key` from supabase-py, read the code and check the version pin BEFORE asking the operator to verify their credentials. The operator's copy-paste is not the problem. Check yourself first.

---

### CRASH-007 — `ModuleNotFoundError: No module named 'websockets.asyncio'` — hard pin blocked dependency upgrade
**Date:** 2026-05-21 22:07 ET  
**Repo:** zerotradingx15minbtc  
**Found by:** Operator (Railway deploy logs)  
**Severity:** crash  
**Status:** fixed  
**Symptom:** `ModuleNotFoundError: No module named 'websockets.asyncio'` on boot. Introduced by CRASH-006 fix.  
**Root cause:** `websockets==12.0` hard pin blocked pip from installing `websockets>=13.0` which `realtime==2.30.0` (transitive dep of `supabase==2.30.0`) requires.  
**Fix:** Loosened `requirements.txt` pins to `>=` minimums. `supabase` stays pinned at `==2.30.0`.  
**Commit:** (next commit in zerotradingx15minbtc)  
**Pattern:** #hardcoded-env-value  
**Cross-repo:** Fix in zerotradingx15minbtc. Same entry in beta CRASH-AND-FIX-LOG.md.

---

### CRASH-008 — `401 Unauthorized` on Kalshi REST — API key environment mismatch
**Date:** 2026-05-21 22:14 ET  
**Repo:** zerotradingx15minbtc  
**Found by:** Operator (Railway deploy logs)  
**Severity:** crash  
**Status:** diagnosed  
**Symptom:** `401 Unauthorized` on `trading-api.kalshi.com/trade-api/v2/portfolio/positions` at boot reconciliation.  
**Root cause:** Demo API key used against production endpoint. Kalshi demo and production keys are completely separate.  
**Fix:** Create a production API key on kalshi.com → Settings → API Keys → Production tab. Update Railway env vars.  
**Pattern:** #paper-live-boundary-confusion  
**Cross-repo:** Fix in zerotradingx15minbtc env config (no code change needed).

---

### CRASH-009 — `401 Unauthorized` — wrong RSA padding scheme (PKCS1v15 instead of PSS)
**Date:** 2026-05-21 22:18 ET  
**Repo:** zerotradingx15minbtc  
**Found by:** Perplexity Computer — cross-referenced working zerotrading-core adapter  
**Severity:** crash  
**Status:** fixed  
**Symptom:** 401 on every Kalshi REST call. Key works fine on zerotrading-core.  
**Root cause:** Phase 1 skeleton used `PKCS1v15`. Kalshi requires RSA-PSS-SHA256, salt_length=32. Documented correctly in zerotrading-core from prior sessions but not carried over to the beta skeleton.  
**Fix:** `padding.PSS(mgf=MGF1(SHA256), salt_length=32)` in `kalshi_rest.py`.  
**Pattern:** #import-contract-mismatch  
**Cross-repo:** Fix in zerotradingx15minbtc. KI-009 added to RAILWAY-KNOWN-ISSUES.md.

---

### CRASH-010 — `401 Unauthorized` — wrong production base URL
**Date:** 2026-05-22 00:58 ET  
**Found by:** Operator — confirmed correct domain via companion repo meszaroszack/zerotrading-core  
**Severity:** crash  
**Status:** fixed  
**Symptom:** `401 Unauthorized` on all authenticated Kalshi REST calls after RSA-PSS fix (CRASH-009). Auth signing is correct.  
**Root cause:** `_BASE_URL` was set to `https://trading-api.kalshi.com/trade-api/v2`. The correct production domain is `https://api.elections.kalshi.com/trade-api/v2`. Confirmed working in `zerotrading-core/src/adapter/kalshi.ts`. Wrong domain always returns 401 regardless of signing correctness.  
**Fix:** Changed `_BASE_URL` production branch in `kalshi_rest.py` from `trading-api.kalshi.com` to `api.elections.kalshi.com`.  
**Commit:** `86ff62b` (beta repo)  
**Pattern:** #wrong-endpoint  

**AGENT NOTE — HARD RULE:**  
Kalshi production REST base URL is `https://api.elections.kalshi.com/trade-api/v2`. Never use `trading-api.kalshi.com`. When a 401 persists after confirming signing is correct, check the base URL against the working implementation in `zerotrading-core/src/adapter/kalshi.ts`.

---

### CRASH-011 — `401 Unauthorized` — wrong message string in RSA-PSS signature
**Date:** 2026-05-22 01:02 ET  
**Found by:** Operator — confirmed via working implementation in `zerotrading-core/src/adapter/kalshi.ts`  
**Severity:** crash  
**Status:** fixed  
**Symptom:** `401 Unauthorized` on all authenticated Kalshi REST calls after base URL fix (CRASH-010).  
**Root cause:** `_sign()` built the signature message using just the short path. Kalshi requires the full path including `/trade-api/v2` prefix. Query string must be stripped before signing.  
**Fix:** `message = (ts_ms + METHOD + "/trade-api/v2" + path.split("?")[0]).encode("utf-8")`  
**Commit:** `3a25f69` (beta repo)  
**Pattern:** #wrong-signature-message  

**AGENT NOTE — HARD RULE:**  
Kalshi signature message = `timestamp_ms + METHOD + /trade-api/v2 + path_without_query_string`. Cross-reference `zerotrading-core/src/adapter/kalshi.ts` for canonical signing logic.

---

### CRASH-012 — Boot crash: wrong `series_ticker` param + `sys.exit(1)` on transient discovery failure
**Date:** 2026-05-22 01:08 ET  
**Found by:** Operator  
**Severity:** crash  
**Status:** fixed  
**Symptom:** Bot crashes at boot either (a) returning no markets because `ticker=KXBTC` is the wrong param for series filtering, or (b) calling `sys.exit(1)` when Kalshi API is momentarily between windows.  
**Root cause (two issues):**  
1. `discover_active_kxbtc15m()` used `{"ticker": "KXBTC"}` — this is a market-level filter. The correct param for filtering by series is `{"series_ticker": "KXBTC15M"}`. Manual KXBTCD exclusion and `startswith("KXBTC-")` filters were only needed to compensate for the wrong param and are now removed.  
2. Boot discovery failure called `sys.exit(1)` — a transient Kalshi blip (market between 15-min windows) would permanently kill the process. The correct behavior is to wait and retry.  
**Fix:**  
1. Changed `_get("/markets", params={"ticker": "KXBTC", ...})` to `params={"series_ticker": "KXBTC15M", ...}`. Removed manual ticker filters.  
2. Wrapped boot discovery in `while True` / `await asyncio.sleep(60)` / `continue` instead of `sys.exit(1)`.  
**Commit:** `8a091a5`  
**Pattern:** #wrong-api-param, #fail-open-on-transient  
**Cross-repo:** Added to parent repo CRASH-AND-FIX-LOG.md.

**AGENT NOTE — HARD RULES:**  
- Kalshi series filter param is `series_ticker`, not `ticker`. Use `series_ticker=KXBTC15M` to get only 15-minute BTC markets.  
- Never `sys.exit()` on a transient discovery failure at boot. KXBTC15M markets roll every 15 minutes — there is a brief window where no market is open. Always retry with sleep.

---

### CRASH-013 — Railway healthcheck failure — HTTP server starts too late
**Date:** 2026-05-22 01:29 ET  
**Found by:** Operator  
**Severity:** crash (Railway marks deploy failed, restarts process)  
**Status:** fixed  
**Symptom:** Railway logs "Healthcheck failure" exactly 10 seconds after deploy. Bot may be fully functional but Railway kills and restarts it because `/health` never returned 200 within the healthcheck window.  
**Root cause:** `start_dashboard()` was called after the full boot sequence — after Supabase init, FSM restore, boot reconciliation, and ticker discovery. Any of these steps hanging (slow DB, Kalshi API blip, between-window discovery retry) delays the HTTP server startup past Railway's 10s healthcheck timeout.  
**Fix:**  
1. `start_dashboard()` moved to the very first line of `run()`, before `validate_env()` and all other boot steps. `/health` returns 200 within ~1 second of process start.  
2. `_cache_loop()` now sleeps 30s before first DB fetch — Supabase is not ready when the dashboard thread starts. The cache serves empty data during the boot window, which is correct behavior.  
3. `register_shared_state()` remains after boot completes, wiring live FSM/feed state into the dashboard once the system is ready.  
**Commit:** `43af109`  
**Pattern:** #healthcheck-timing  
**Cross-repo:** Added to parent repo CRASH-AND-FIX-LOG.md. Added to RAILWAY-KNOWN-ISSUES.md as KI-013.

**AGENT NOTE — HARD RULE:**  
The HTTP server (`start_dashboard()`) must be the FIRST thing called in `run()`. Never move it after any async boot step. Railway's healthcheck does not wait for your boot sequence. The dashboard is designed to serve partial/empty state during boot — this is intentional and correct.

---

### CRASH-014 — Dashboard shows UNKNOWN/DISCONNECTED/blank — state wiring broken in 5 places
**Date:** 2026-05-22 01:35 ET  
**Found by:** Operator (dashboard live but showing no data)  
**Severity:** major (dashboard non-functional, bot still trades)  
**Status:** fixed  
**Symptom:** Dashboard up, /health 200, but FSM shows UNKNOWN, feeds show DISCONNECTED, BTC spot blank, log tail empty.  
**Root cause (5 independent issues):**  
1. `_LogBufferHandler` was wired inside a function (`_wire_log_handler()`) — if health.py was imported late, early log lines were missed. Fix: attach to root logger at module level, at import time.  
2. `health.py` was not imported before first logging call in main.py — the log buffer missed all boot logs. Fix: `import src.ops.health as _health_module` at top of main.py.  
3. `register_shared_state()` used `asyncio.get_event_loop()` which returns the wrong loop if called from inside a running coroutine. Fix: `asyncio.get_running_loop()` first, fall back to `get_event_loop()`.  
4. Feed health fallback was missing the correct attribute path `.kalshi.is_live` / `.binance.is_live` (the nested FeedStatus dataclass). Fix: try `_feed_health.kalshi.is_live` as primary fallback.  
5. BTC spot read assumed `btc_spot` was always a list — not defensive against future refactor. Fix: `isinstance` check for list/tuple/scalar.  
**Fix:** 5 targeted edits across `src/ops/health.py`, `src/ops/dashboard.py`, `main.py`.  
**Commit:** `a980810`  
**Pattern:** #state-wiring, #import-order  
**Cross-repo:** Added to parent repo CRASH-AND-FIX-LOG.md.

**AGENT NOTE — HARD RULES:**  
- `_LogBufferHandler` must be attached at module level (not inside a function) so it fires on import.  
- `import src.ops.health` must be the first import in main.py after stdlib, before any other project module.  
- Use `asyncio.get_running_loop()` inside a coroutine context, not `get_event_loop()`.  
- `ExecutionFSM` has NO in-memory `_state`. It is DB-backed. `current_state()` is always a Supabase query.  
- `FeedHealthMonitor.status_dict()` is the canonical read path. Fallback is `.kalshi.is_live` / `.binance.is_live` on the nested `FeedStatus` dataclass.  
- `SharedState.btc_spot` is `list[float]` — read as `btc_spot[0]`.

---

### CRASH-015 — Dashboard shows UNKNOWN FSM state — coroutine call from sync HTTP thread
**Date:** 2026-05-22 01:46 ET  
**Found by:** Operator  
**Severity:** major (dashboard shows stale/UNKNOWN data despite bot trading correctly)  
**Status:** fixed  
**Symptom:** Dashboard FSM state stuck on UNKNOWN, BTC spot blank, active ticker blank. `run_coroutine_threadsafe` either returns before DB responds or holds a stale event loop reference.  
**Root cause:** `_build_status()` runs in the stdlib HTTP server thread. It attempted to call `_fsm.current_state()` via `run_coroutine_threadsafe()`. `ExecutionFSM.current_state()` is a pure DB call — it has no in-memory state. From a sync thread, the coroutine call is unreliable and races against the event loop's DB round-trip. No caching of last-known state meant UNKNOWN was returned whenever the call timed out.  
**Fix:** Introduced `push_state(fsm_state, btc_spot, active_ticker)` — a sync function called from the async trading loop immediately after `await fsm.current_state()`. It writes three Python primitives to module-level vars. `_build_status()` reads those vars directly — zero coroutines, zero thread-safety issues, zero async calls from the HTTP thread. Feed health reads `FeedHealthMonitor.status_dict()` which uses only `time.time()` math — always safe from any thread.  
**Commit:** `ad129b6`  
**Pattern:** #sync-async-boundary  
**Cross-repo:** Added to parent repo CRASH-AND-FIX-LOG.md.

**AGENT NOTE — HARD RULE:**  
Never call async coroutines from a sync thread (HTTP handler, background thread) to read bot state. The pattern is: async loop calls `push_state()` after every `await fsm.current_state()`, HTTP thread reads the cached primitives. `FeedHealthMonitor.status_dict()` is sync-safe (pure time math). `ExecutionFSM.current_state()` is NEVER sync-safe — it always hits Supabase.

---

### CRASH-016 — Silent boot crash: uncaught httpx.HTTPStatusError from boot_reconcile
**Date:** 2026-05-22 01:58 ET  
**Found by:** Perplexity Computer — read reconcile.py and main.py after inspecting live /api/status  
**Severity:** crash (boot loops every ~6 minutes, dashboard stays in perpetual UNKNOWN state)  
**Status:** fixed  
**Symptom:** Dashboard live at /health 200, but fsm_state=UNKNOWN, btc_spot=0, feeds DISCONNECTED, uptime resets every 5–6 minutes. Bot never reaches _paper_loop.  
**Root cause:** `boot_reconcile()` calls `await rest.get_positions()` with no try/except. Any Kalshi REST error (401, 403, 5xx, connection timeout) raises `httpx.HTTPStatusError`. In `main.py`, the `except ReconcileError` block only catches `ReconcileError` — an `httpx.HTTPStatusError` escapes uncaught, crashes `run()`, and Railway restarts the process. Every restart hits the same error, creating an infinite restart loop. The bot process was live (dashboard answering /health) but never surviving past boot_reconcile.  
**Fix:**  
1. Wrapped `rest.get_positions()` in `boot_reconcile()` with `try/except Exception` that converts any transport error into `ReconcileError`.  
2. Changed `main.py` boot reconcile block from single `try/except → sys.exit(2)` to `while True` retry loop. Transport failures retry every 30s. Only ORPHANED (logic error requiring manual review) calls `sys.exit(2)`.  
**Commit:** `abd7478`  
**Pattern:** #uncaught-exception, #boot-loop  
**Cross-repo:** Added to parent repo CRASH-AND-FIX-LOG.md.

**AGENT NOTE — HARD RULE:**  
Every `await rest.*()` call in boot_reconcile and anywhere in the boot sequence must be wrapped in try/except. An unhandled httpx error before the trading loop starts will silently crash the process. The Railway restart loop symptom is: dashboard at /health 200, uptime resets every few minutes, fsm_state always UNKNOWN.

---

### CORE-STABILIZATION-V1 — zerotrading-core PR (BUG-1..BUG-6, UI-1, UI-2)
**Date:** 2026-05-23 ET
**Repo:** github.com/meszaroszack/zerotrading-core
**Branch:** fix/core-stabilization-v1 (off core-engine-review-v1)
**Severity:** mixed (1 minor + 6 major + 1 crash class)
**Status:** PR open (not auto-merged per operator rule)
**Cross-repo:** Full per-entry details live in `zerotrading-core/ai/summaries/CRASH-AND-FIX-LOG.md` (FIX-CORE-001..FIX-CORE-008).

**Summary of fixes:**
1. **BUG-1 / FIX-CORE-001 (`c2f4f33`)** — Settlement P&L now booked via new `MARKET_SETTLED` event + watcher + `ACCT_BOOK_SETTLEMENT_PNL` ledger effect. Realized P&L lives in the ledger layer (per the architecture correction); UI derives from join.
2. **BUG-2 / FIX-CORE-002 (`c8c53bd`)** — Time-exit fallback no longer collapses to `Infinity` when `watchedMarket` is undefined. Eslint `no-restricted-syntax` rule blocks the regression.
3. **BUG-3 / FIX-CORE-003 (`2d3091b`)** — Urgent exits now log structured `urgent_exit_*` lines, reprice side-aware against the contra side, and escalate to IOC after N failed reposts.
4. **BUG-4 / FIX-CORE-004 (`531b48e`)** — Crash artifact (`crash.json`) + transient error filter (ECONNRESET/ETIMEDOUT/EAI_AGAIN/ENOTFOUND/ECONNREFUSED/EPIPE/kalshi_rate_limited) + periodic `health.json` writer + `no-floating-promises` audit.
5. **BUG-5 / FIX-CORE-005 (`ebff8ad`)** — `STATE_DIR` env wired through all file persistence; `railway.json` declares `zt-core-state` volume at `/data`; `docs/RAILWAY-DEPLOY.md` and `docs/TODO-SUPABASE-MIGRATION.md` added.
6. **BUG-6 / FIX-CORE-006 (`2c3e910`)** — `/api/status.balance` served from a TTL cache (8s) with 5s background refresh and stale-on-error. 50 reads → 1 fetcher call.
7. **UI-1 / FIX-CORE-007 (`140acb1`)** — `/api/status.tradeHistory` now joins positions ⨝ ledger by `cycleId/ticker` (Invariant I-15: confirmed overrides estimated). Adds `exitMode='settled_hold'` for closed-by-resolve cycles. KPI summary: cyclesWon / cyclesLost / cyclesHeld / totalNetPnlCents / totalFeesCents / todayNetPnlCents / winRate.
8. **UI-2 / FIX-CORE-008 (`bd24cdb`)** — Bid/ask + BTC spot ring buffer (capacity 600, persisted to `STATE_DIR/bid-series.json`, sampler state-guarded on ENTERING/OPEN/EXITING). Dashboard sparkline renders NO bid (green), NO ask (red), BTC spot (dashed gray).

**Acceptance:** typecheck clean, 54/54 tests pass across 7 files, lint reports 0 errors and 4 pre-existing unused-var warnings. Strategy thresholds, regime weights, and sizing were **not** changed (operator constraint).

**Railway service config (operator action required, not changed via API):**
- `STATE_DIR=/data`
- Volume `zt-core-state` mounted at `/data`
- Source branch = `main`

**Pattern:** #event-coverage, #ledger-join, #crash-artifacts, #persistence-contract, #cache-with-staleness, #ring-buffer

---

### CORE-STABILIZATION-V1-RECONCILE — Branch divergence: agent worked off a stale base
**Date:** 2026-05-23 04:57 ET
**Repo:** github.com/meszaroszack/zerotrading-core (process bug; affects governance for all repos)
**Found by:** Perplexity Computer — discovered when attempting `gh pr merge 1` returned "not mergeable" due to conflicts
**Severity:** silent-wrong (process) — could have shipped duplicated/conflicting fixes and lost 18 commits of work on main
**Status:** fixed (reconciled via cherry-pick onto main; merged as `694f4df`)

**Symptom:**
- PR #1 (`fix/core-stabilization-v1`) was based on `core-engine-review-v1`, a named feature branch that was 18 commits behind `origin/main`.
- `main` had independently received `7ea6400` "full-functional-repair: all 9 bugs fixed, 30 tests" which solved an overlapping set of bugs (BUG-1 settlement P&L, BUG-2 time-exit) using a different architecture (e.g., main removed time-exit entirely in `be99ad2`; agent's PR fixed time-exit fallback).
- Direct merge of PR #1 would either A) overwrite main's superior fixes, or B) double-implement the same logic with conflicting contracts (PositionRecord vs ledger-join for tradeHistory; settlement watcher vs `GET /markets/{ticker}` polling).

**Root cause:**
- Agent skipped the **MASTER-PROMPT v2 startup sequence**: did not run `git fetch && git log origin/main..HEAD` to verify branch base, did not read `ai/handoffs/CURRENT-STATE.md` which clearly stated `Latest commit: f4b7bdc` and "Time-exit intentionally removed — do not re-add".
- Agent branched off whatever was checked out locally (`core-engine-review-v1`) without confirming it was current with `origin/main`.
- MASTER-PROMPT required these steps but had no machine-readable enforcement — no AGENTS.md, no startup-checklist file, no PR template, no branch-protection rule requiring up-to-date-with-base.
- No drift guardrail caught the 18-commit gap before code was written.

**Fix:**
1. **Code reconciliation** — Created `reconcile/core-stabilization-v1` off `origin/main`, cherry-picked only the bug fixes that are still relevant on main (BUG-3 side-aware urgent exit, BUG-4 crash artifact + health.json, BUG-5 STATE_DIR + Railway volume, BUG-6 balance cache, UI-2 bidSeries sparkline, plus eslint scaffold). Dropped BUG-1, BUG-2, UI-1 — main already solved them or solved them differently. Hand-resolved every conflict (UI-2 kept main's PositionRecord tradeHistory and only added the `bidSeries` line; BUG-3 dropped `timeExitTriggered` references since main removed time-exit entirely). Validated `typecheck` clean, 68/68 tests passing, lint 0 errors. Closed PR #1, opened PR #2, merged as `694f4df`. Railway auto-redeploys from main.
2. **Process guardrails** — Added three required-reading documents in parent repo:
   - `AGENTS.md` (root) — auto-read by Cursor/Claude Code/Copilot/Codex/Perplexity. Seven hard rules incl. "branch from `origin/main` — never a named feature branch", "main must always be deployable", "no force-push on shared branches", "fast-forward or rebase before opening PR".
   - `ai/guardrails/BRANCH-AND-MERGE-DISCIPLINE.md` — required-reading guardrail with the worked-example reconstruction of this incident.
   - `ai/checklists/SESSION-STARTUP-CHECKLIST.md` — 60-second copy-paste shell block (`git fetch && git status && git log --oneline -10 origin/main..HEAD && gh pr list && read CURRENT-STATE.md`) plus the four questions every agent must answer before writing any code.

**Operator follow-ups required (cannot be done via API):**
- Enable branch protection on `zerotrading-core` and `zerotrading` `main` branches: require PR review, require linear history (rebase or squash only), require status checks (typecheck + tests + lint) green, require branch to be up-to-date before merge.
- Delete stale branches: `core-engine-review-v1`, `fix/core-stabilization-v1`, `reconcile/core-stabilization-v1` (kept locally for archaeology, removable once team confirms).

**Pattern:** #stale-base, #missing-startup-discipline, #governance-not-enforced
**Cross-repo:** This is a process bug, not a code bug. Applies to ALL ZeroTrading repos (parent, core, x15minbtc) and to any future repo that ships under the same AI-first contract.

**AGENT NOTE — HARD RULE:**
Before writing a single line of code in any ZeroTrading repo:
1. `cd <repo> && git fetch origin && git status && git log --oneline -10 origin/main..HEAD`
2. If `HEAD` is not on `origin/main` or is behind, STOP. Either rebase onto `origin/main` or branch fresh from `origin/main`. Never branch off a named feature branch.
3. Read `ai/handoffs/CURRENT-STATE.md`. Confirm `Latest commit:` matches `git log -1 origin/main --format=%h`. If it does not, ask the operator before doing anything else.
4. `gh pr list --state open` — check for in-flight work that overlaps your task.

---

### FIX-CORE-009 — Dashboard blank in production: ReferenceError: posBadgeColor is not defined
**Date:** 2026-05-23 01:30 ET
**Repo:** github.com/meszaroszack/zerotrading-core
**Found by:** Operator (screenshot of blank dashboard at https://zerotrading-core-production.up.railway.app); confirmed in headless browser by Perplexity Computer
**Severity:** silent-wrong (UI completely blank; bot continues trading correctly server-side, so failure is invisible from API/health checks)
**Status:** fixed (PR #3 squash-merged as `06173c2`; Railway auto-redeployed; verified live — 9 cards rendering, zero JS errors)

**Symptom:**
- Dashboard URL loaded: HTML 200, CSS 200, JS bundle 200, `/api/status` returned full 47kB JSON payload showing position OPEN, 2 contracts, uptime ~29min.
- Page rendered as solid dark background — React root `<div id="root">` had zero child nodes.
- Console error at first render: `ReferenceError: posBadgeColor is not defined`. Whole React tree failed to mount.

**Root cause:**
- Commit `7ea6400` ("full-functional-repair: all 9 bugs fixed, 30 tests passing, zero TS errors") deleted both `function posBadgeColor(state)` and `function renderExitPosture(ep)` from `client/src/Dashboard.tsx` but left their call sites intact at lines 421 and 480.
- The bundle still compiled because:
  1. Root `npm run typecheck` script only invokes `tsc --noEmit` against the **server** `tsconfig.json`. The client lives under `client/tsconfig.json` and was never wired into the root script. `tsc -p client/tsconfig.json` does flag both errors — but it was never run in CI or in the pre-merge gate.
  2. Vite production build does not enforce TypeScript reference resolution inside JSX template-string interpolations (`className={`badge ${posBadgeColor(state)}`}`). It emits the call into the bundle as-is and fails only at runtime.
- The commit message claimed "zero TS errors" — which was true for the server tsconfig that was actually checked. The client tsconfig was never checked.

**Fix:**
1. Restored both helpers verbatim from their pre-`7ea6400` definitions, with doc comments pointing at this incident (`fix(ui): restore posBadgeColor + renderExitPosture`, merged as `06173c2`).
2. Validation gates that should be added (separate PR, tracked in CURRENT-STATE follow-ups):
   - Add `tsc -p client/tsconfig.json` to the root `typecheck` script so `npm run typecheck` covers both halves of the repo.
   - Add a smoke test that boots the served bundle in a headless browser and asserts `document.getElementById('root').children.length > 0`.
3. Branch protection (operator action, still pending) must require `typecheck` green before merge — currently only enforced by convention.

**Commit:** `06173c2` (on `meszaroszack/zerotrading-core` main)
**Pattern:** #partial-checks, #ts-not-enforced-in-jsx-interp, #ui-only-failure-invisible-to-health-probes
**Cross-repo:** This entry. The child repo entry will appear in `zerotrading-core/ai/summaries/CRASH-AND-FIX-LOG.md` once that scaffold is created (separate session).

**AGENT NOTE — HARD RULE:**
"Typecheck passes" is only meaningful if the script actually checks every TypeScript subtree in the repo. Before claiming a build is green, run `find . -name "tsconfig*.json" -not -path "*/node_modules/*"` and confirm every config is invoked. For React/Vite projects specifically: prod build does NOT catch ReferenceErrors inside JSX template interpolations — only `tsc --noEmit` does. A passing build is not a passing typecheck.

---

### FIX-CORE-010 — Exit logic silently disabled + dashboard fields blank when orderbook REST returns null
**Date:** 2026-05-23 13:50 ET
**Repo:** github.com/meszaroszack/zerotrading-core
**Found by:** Operator (watching live dashboard show `—` for avg entry, current bid, yes ask, unrealized P&L despite holding 2 contracts on `KXBTCD-26MAY2314-T75399.99`); root cause traced by Perplexity Computer.
**Severity:** silent-wrong (exit logic functionally broken; dashboard misreports state; bot relied on hour-close settlement to rescue positions when REST orderbook was momentarily unavailable. Operator confirmed: "P&L math has never really worked.")
**Status:** fixed (PR #4 squash-merged as `f809f18`; PR #5 squash-merged as `6e495cf`; Railway auto-redeployed; verified live — `bootedAtMs` advanced, `currentNoBidCents` rendering real data, `position.state: IDLE` after operator cleared Kalshi.)

**Symptom:**
- Right-hand side of dashboard rendered `—` for: avg entry price, current NO bid, YES ask, unrealized P&L per contract, unrealized P&L total.
- `/api/status` payload confirmed `position.exitPosture: null` AND `position.avgEntryPriceCents: null` AND `position.currentNoBidCents: null` even though the bot had an open position with a fully resolved fill history (`tradeHistory.entryAvgPrice: "0.5800"`).
- Less visible but more dangerous: `URGENT_EXIT_TICK` audit logs were missing for one or more strategy ticks per blip — meaning TP/SL trigger logic also never ran during the same window. The bot had no functional exit path during a REST blip; only settlement at hour-close rescued the position. Operator confirmed this behavior has been present "in every build of this model."

**Root cause:**
- `src/strategy/endOfHour.ts` `handleOpen()` line 605 early-returned whenever `adapter.getOrderbook(ticker)` returned `null` OR `orderbook.noBid <= 0`. On early-return, `snap.exitPosture` stayed `null` for that tick.
- `src/server/index.ts` line 259 built the API position object by reading ALL UI display fields from `snap.exitPosture?.X ?? null`. Single source of truth → single point of failure. When the orderbook was momentarily unavailable, the entire right column collapsed.
- TP/SL trigger logic at lines 726–729 lived **inside the same gated block** — so the same blip that blanked the UI also silently disabled exit logic. There was no second code path or fallback.
- The same file's `scoreCandidate()` at line 816 ALREADY implemented the correct fallback pattern: `(orderbook?.noBid || 0) > 0 ? orderbook!.noBid : market.no_bid_cents`. Entry-decision logic was robust; exit-decision logic was not. This inconsistency is the bug.
- Reference repos in operator's GitHub account (`kalshi-btcd-trader/server/tradingEngine.ts:2383-2445`, `kalshi-15m-bot/src/feed/ws_feed.py:123`) all use a fallback or dual-source pattern for current price. The pattern was known internally but had not been propagated to `zerotrading-core`.

**Fix:**
Split into two PRs for blast-radius safety, both merged to main same session:

1. **PR #4 (`f809f18`) — `fix(ui): show position fields when orderbook is unavailable`** (DISPLAY ONLY)
   - Extracted exported pure helper `derivePositionUiFields()` in `src/server/index.ts`.
   - Source priority for UI fields:
     - `avgEntryPriceCents` ← `exitPosture`, else parse `activePos.entryAvgPrice` (the persisted fill-derived dollar string).
     - `currentNoBidCents` ← `exitPosture`, else `snap.watchedMarket.noBidCents` (from `getCurrentHourMarkets`).
     - `yesAskCents` ← `exitPosture`, else `snap.watchedMarket.yesAskCents`.
     - `unrealizedPnl*` ← `exitPosture`, else computed inline from above.
   - 7 colocated tests at `src/server/derivePositionUiFields.test.ts`.
   - Does NOT change trading behavior.

2. **PR #5 (`6e495cf`) — `fix(strategy): TP/SL fallback when orderbook REST returns null`** (TRADING BEHAVIOR)
   - New exported helper `synthesizeOrderbookFromSummary(summary, fetchedAt)` builds a minimal `KalshiOrderbookView` from a `KalshiMarketSummary`'s top-of-book fields.
   - `handleOpen()` restructured: market summaries fetched FIRST (was already fetched later, just moved up), THEN orderbook attempted, THEN fallback synthesized from summary if REST returned null/empty.
   - New gate: enter the market-finalized reconcile branch ONLY when both sources are unusable (REST null AND summary status closed/settled OR `no_bid_cents == 0`). This preserves the correct hour-close settlement path.
   - `URGENT_EXIT_TICK` audit log now includes `orderbookFromFallback: boolean` for operator visibility. Expected to be `false` in steady state; non-zero rate indicates REST instability worth investigating.
   - 7 colocated tests in `src/strategy/endOfHour.test.ts` (file went 13 → 20 tests; total suite 68 → 82).

**Commits:** `f809f18` (PR #4), `6e495cf` (PR #5)
**Pattern:** #single-source-coupling, #happy-path-only, #invisible-trading-failure, #pattern-existed-in-sibling-not-propagated
**Cross-repo:** Parent (this entry). Child `zerotrading-core/ai/` scaffold still pending — when created, this entry's body will be mirrored verbatim into `zerotrading-core/ai/summaries/CRASH-AND-FIX-LOG.md`.

**AGENT NOTE — HARD RULE:**
Any code path that reads "current price" or "current bid/ask" from a single live REST source is wrong by default. Always read from at least two sources (live + cached summary) and synthesize a fallback. The bar is: "if Kalshi REST returns 500 for 30 seconds, does TP/SL still fire?" If the answer is "we'd wait for the next tick," that is the bug. Also: when adding a new piece of decision logic that depends on market data, ALWAYS grep the same file for the word "orderbook" to confirm whether a fallback pattern already exists nearby — `scoreCandidate` had it for 6 weeks before this bug was found.
