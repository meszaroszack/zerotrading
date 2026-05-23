# Session Summary — 2026-05-23 (Core stabilization v1 reconcile + governance pack)

**Window:** ~2026-05-22 evening to 2026-05-23 01:00 ET
**Operator:** meszaroszack (Zack)
**Agent:** Perplexity Computer
**Repos touched:** `meszaroszack/zerotrading-core`, `meszaroszack/zerotrading` (parent)
**Outcome:** Core stabilization PR reconciled with `main` and merged (`694f4df`). Three new governance documents added to parent repo to prevent the branch-divergence incident from recurring.

---

## What we set out to do

User request: "merge to main so railway updates - then do a big documentation and research push with all of your learnings - we want to save water and usage in the future again, so we cant go backwards and make the same mistakes - there can be no drift."

Specifically:
1. Land the eight-bug core stabilization work (PR #1 on `zerotrading-core`).
2. Audit the parent repo's master prompts / governance for gaps.
3. Write down "don't repeat this" lessons so neither humans nor future agents recreate the same problems.

---

## What actually happened

### 1. PR #1 was not mergeable

`gh pr merge 1` returned "not mergeable." Investigation:

- PR #1 branch (`fix/core-stabilization-v1`) was based on `core-engine-review-v1`.
- `core-engine-review-v1` was **18 commits behind** `origin/main`.
- `origin/main` had `7ea6400` "full-functional-repair: all 9 bugs fixed, 30 tests" — an independent fix pass that overlapped 3 of the 8 bugs in PR #1, sometimes with a different architectural choice (e.g., main *removed* time-exit entirely in `be99ad2`; PR #1 *fixed* time-exit's `Infinity` fallback).

Direct merge would either lose main's 18 commits of independent work or double-implement overlapping logic with conflicting contracts.

### 2. User decision

Asked the user how to proceed. Options were (a) reset main to PR branch, (b) merge main into PR and resolve, (c) cherry-pick PR work onto main, dropping duplicates. User answered:

> "if yours is fully functional - then lets run with that - put notate and make clear for future humans and agents what has gone wrong and correct instructions so that nothing gets left in that state without better explanation things should all be merged to main at the end of sessions"

Translation: don't blindly take PR; favor what is functional on main, cherry-pick the rest, document everything that went wrong. That meant option (c) — cherry-pick.

### 3. Reconciliation execution

Created `reconcile/core-stabilization-v1` off `origin/main`. Cherry-picked in this order with manual conflict resolution at each step:

| Commit | Change | Conflict notes |
|---|---|---|
| `46e75aa` | BUG-5: `STATE_DIR` + Railway volume + docs | clean |
| `e846cf3` | BUG-6: balance cache | clean |
| `d523670` | BUG-4: crashLog + health.json | clean |
| `0f2bf83` | BUG-3: side-aware urgent exit | dropped all `timeExitTriggered` references — main removed time-exit in `be99ad2` |
| `6008153` | UI-2: bidSeries sparkline | kept main's PositionRecord-outcome `tradeHistory` mapping; only added the `bidSeries: deps.bidSeriesStore.tail(300)` line into `/api/status` |
| `505ddd9` | eslint scaffold | clean |

**Dropped** (already solved differently on main):
- BUG-1 settlement P&L — main uses `GET /markets/{ticker}` polling; PR used a `MARKET_SETTLED` watcher + ledger event.
- BUG-2 time-exit `Infinity` fallback — main removed time-exit entirely.
- UI-1 tradeHistory join — main writes outcome to `PositionRecord` directly; no ledger join.

### 4. Validation

- `npm run typecheck` — clean
- `npm test` — 68/68 passing across 5 files (30 from main's `core.test.ts` + 38 from agent's colocated suites: bidSeries 7, balanceCache 5, crashLog 13, endOfHour 7, scaffold 1, plus position/etc).
- `npm run lint` — 0 errors, 11 pre-existing unused-var warnings.

### 5. Merge

- Closed PR #1 (unmerged).
- Pushed `reconcile/core-stabilization-v1` to origin.
- Opened PR #2.
- Merged PR #2 as `694f4df`. Railway auto-redeploys from `main`.

### 6. Governance pack

Audited the parent repo's existing `ai/` scaffold. It is genuinely mature — MASTER-PROMPT v2 exists, MODEL-SPEC and REPO-PRINCIPLES exist, append-only logs exist, `.cursor/rules/` exist. The problem is that **the rules were not enforced** and were **not easy to find on a 60-second skim**.

Added three documents to close the gap:

1. **`AGENTS.md`** (repo root) — every modern AI tool (Cursor, Claude Code, Copilot, Codex, Perplexity) auto-reads a file named `AGENTS.md` if it sits at the repo root. Contains seven hard rules including "branch from `origin/main` — never a named feature branch," "main must always be deployable," and "no force-push on shared branches," plus the repository map and end-of-session checklist.
2. **`ai/guardrails/BRANCH-AND-MERGE-DISCIPLINE.md`** — required-reading guardrail with the 60-second startup shell block, the four pre-flight questions, the trunk-based-development model, and the worked-example reconstruction of this incident.
3. **`ai/checklists/SESSION-STARTUP-CHECKLIST.md`** — copy-paste-able shell block + four questions every agent must answer before writing code, plus the list of anti-patterns to recognize.

Updated:
- `ai/summaries/DECISION-LOG.md` — appended the 2026-05-23 reconciliation entry.
- `ai/summaries/CRASH-AND-FIX-LOG.md` — appended `CORE-STABILIZATION-V1-RECONCILE` entry.
- `ai/handoffs/CURRENT-STATE.md` — refreshed to `Latest commit: 694f4df`, added required-reading section, listed operator follow-ups.

---

## Open operator actions (cannot be done via API)

1. **Enable branch protection** on `meszaroszack/zerotrading-core` and `meszaroszack/zerotrading` `main` branches: require PR review, linear history, status checks green, up-to-date with base before merge.
2. **Delete stale branches** on `zerotrading-core`: `core-engine-review-v1`, `fix/core-stabilization-v1`, `reconcile/core-stabilization-v1`.

---

## Lessons (added to guardrail docs in detail)

1. **`git fetch && git log origin/main..HEAD` is non-negotiable** before writing any code.
2. **Branch from `origin/main`** — never from a named feature branch. Stale-base divergence is silent and only surfaces at merge.
3. **`CURRENT-STATE.md` must be read every session** and its `Latest commit:` line must match `git log -1 origin/main --format=%h`.
4. **Document divergence the moment it is discovered** — don't try to merge through it.
5. **Process rules need machine-readable enforcement** — branch protection on GitHub, AGENTS.md at repo root, checklists that fit in 60 seconds.

---

## What's next (per operator)

User has teed up a follow-on conversation:

> "yes finish the merge to main - deliver the documentation packet - crash decision logs etc - then we will discuss the big github structural changes and saas values - have multiple githubs we need to do this on and then a number that are more 'research' where i dont feel the need to go that far - but need to layer in some sort of instructions as well."

Plan for next session:
1. Operator provides the full repo list (production vs research).
2. Define a **two-tier governance template**: full (`ai/` scaffold + AGENTS.md + checklists + guardrails + branch protection) for production repos, lite (`AGENTS.md` + minimal `ai/handoffs/CURRENT-STATE.md` + `CHANGELOG.md`) for research repos.
3. Apply each tier to the relevant repos in one pass.
