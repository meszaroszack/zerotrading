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
