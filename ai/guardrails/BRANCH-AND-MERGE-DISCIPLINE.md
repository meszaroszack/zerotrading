# Branch and Merge Discipline

**Required reading for every agent and human contributor.** This guardrail is the direct response to the 2026-05-23 reconciliation incident where two independent fix efforts ran on parallel branches and one of them had to be cherry-picked and re-merged after the fact.

If a 60-second check at the start of a session is the cost of preventing a 60-minute reconciliation at the end, take the 60 seconds.

---

## The non-negotiable startup checklist

Before writing a single line of code in any ZeroTrading repo, complete every step. No exceptions.

1. **`git fetch --all --prune`** in every repo you may touch this session.
2. **`git log origin/main -5 --oneline`** — read the last five commits on `main`. If you don't recognize a commit, read its full message before continuing.
3. **Read `ai/handoffs/CURRENT-STATE.md` in the parent repo (`zerotrading`).** Match the `Latest commit:` line against what you see in step 2. If they disagree, CURRENT-STATE is stale — refresh it before any other work.
4. **Read the most recent dated file in `ai/summaries/`** (highest filename wins). This is the operator's last known position.
5. **`gh pr list --state open`** in every repo. If an open PR touches a file you plan to touch, stop and read that PR first.
6. **Branch from `origin/main`, never from a named feature branch** (e.g., `core-engine-review-v1`, `fix/anything`). Named branches drift. `main` is the only branch guaranteed to match Railway.

   ```bash
   git fetch origin main
   git checkout -b fix/<short-name> origin/main
   ```

7. **State your starting commit in the session output.** Example: `Branched from origin/main @ 7ea6400`. This is your alibi if main moves under you mid-session.

---

## The non-negotiable end-of-session checklist

Before declaring "done" in any session, complete every step.

1. **Open a PR against `main`** — never push directly. Even a one-line typo fix.
2. **Run `npm run typecheck && npm test && npm run lint`** locally. Paste the output in the PR body.
3. **Update `ai/handoffs/CURRENT-STATE.md`** in the parent repo. Bump the `Latest commit:` line. Note any "Do Not Regress" items the session added.
4. **Append to `ai/summaries/DECISION-LOG.md`** if the session made any architectural choice.
5. **Append to `ai/summaries/CRASH-AND-FIX-LOG.md`** if any bug was found or fixed.
6. **Write a dated summary** in `ai/summaries/YYYY-MM-DD-HHMM-<short-name>.md` (this is the file the next agent will read first).
7. **Merge the PR to `main`** so Railway redeploys. Do not leave PRs open across sessions unless the operator explicitly asks you to.

---

## How the 2026-05-23 incident happened

A factual reconstruction, intended as the canonical worked example. Future agents should be able to read this and immediately see how to avoid the same trap.

**Setup:**
- `zerotrading-core` had two long-lived branches: `main` (deployed to Railway) and `core-engine-review-v1` (a feature branch that had served as the integration branch for an earlier multi-bug fix effort).
- The operator briefed agent A with a task spec that named `core-engine-review-v1` as the source branch.
- Independently, agent B was working on `main` and pushed 18 commits over several days, including `7ea6400` "full-functional-repair: all 9 bugs fixed, 30 tests passing".
- Agent A skipped the startup checklist — never read CURRENT-STATE, never fetched `origin/main`, never listed open PRs.

**Failure mode:**
- Agent A branched `fix/core-stabilization-v1` from `core-engine-review-v1`, did roughly 10 hours of work, opened a PR against `main`, and discovered the merge was not mergeable.
- The conflict was not cosmetic. Agent A's "BUG-2: finite time-exit fallback" directly contradicted agent B's `be99ad2` "remove time-based exit trigger from strategy". Agent A's "BUG-1: settlement P&L via MARKET_SETTLED event + watcher" overlapped with agent B's `d87cfe0` "settlement detection rewrite — use GET /markets/{ticker} status".
- Reconciliation required ~90 minutes of careful per-commit triage to keep main's bugfixes while cherry-picking agent A's non-overlapping value (BUG-3, BUG-4, BUG-5, BUG-6, UI-2).

**Root cause:**
- A single missed step at the start of the session (read CURRENT-STATE, fetch main, list PRs) caused the entire downstream rework.
- The MASTER-PROMPT already required this step. The startup checklist was not enforced — there was no concrete consequence for skipping it, and no checkbox before code-writing began.

**What this guardrail adds:**
- The exact checklist above, written in shell-command form so the operator and agents can copy-paste.
- The explicit rule "branch from `origin/main`, never from a named feature branch."
- A worked example of the failure mode, so future agents can pattern-match before repeating it.

---

## The "stale branch" trap

Long-lived feature branches are a common source of this failure mode. Once a feature branch has been merged to `main`, treat it as archival. Specifically:

- **Never branch a new fix off a feature branch.** Always branch off `origin/main`.
- **Delete merged feature branches** within a week of merging. Stale branches are an attractive nuisance — they invite new agents to think they're current.
- **If a branch must be kept open across sessions**, prefix it with `wip/` and document in CURRENT-STATE what's in flight.

The branches alive in `zerotrading-core` as of this guardrail's authoring:

| Branch | Status | Action |
|---|---|---|
| `main` | Live, Railway deploys from here | Keep, protect |
| `core-engine-review-v1` | Stale (last touched pre-`7ea6400`) | Delete |
| `fix/core-stabilization-v1` | Superseded by `reconcile/core-stabilization-v1` | Delete |
| `reconcile/core-stabilization-v1` | Merged via PR #2 | Delete after this guardrail commits |

---

## "What if I don't have time for all this?"

You do. The check is six shell commands and takes less than a minute. If a session is too rushed for that, the session is too rushed to be writing production code on a live trading bot.

---

## Required reading for every fresh session

1. `ai/prompts/MASTER-PROMPT.md`
2. **`ai/guardrails/BRANCH-AND-MERGE-DISCIPLINE.md`** (this file)
3. `ai/handoffs/CURRENT-STATE.md`
4. The most recent dated summary in `ai/summaries/`
5. `ai/summaries/CRASH-AND-FIX-LOG.md`
6. `ai/guardrails/PERSISTENCE-RECONCILIATION.md`
