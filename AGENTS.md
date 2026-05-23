# AGENTS.md — Hard Rules for AI Contributors

Conventional `AGENTS.md` lives at the repo root so every AI tool (Cursor, Claude Code, Copilot, Codex, Perplexity Computer, etc.) sees it without being told.

**This file is the short version. The long version is `ai/prompts/MASTER-PROMPT.md`. If you have time for only one file, read this one. If you have time for a second, read MASTER-PROMPT.**

---

## The seven hard rules

These rules are not suggestions. Violating any of them is a regression that another agent or the operator will have to clean up.

### 1. Read state before writing code

Before any code change, in this exact order:

1. `git fetch --all --prune`
2. `git log origin/main -5 --oneline` — read it
3. `ai/handoffs/CURRENT-STATE.md` in `zerotrading-parent`
4. The newest file in `ai/summaries/`
5. `gh pr list --state open`

If you skip this and ship a fix that conflicts with main, the operator will ask why, and "I forgot" is not an answer that survives twice.

### 2. Branch from `origin/main`, never from a named feature branch

```bash
git fetch origin main
git checkout -b fix/<short-name> origin/main
```

Feature branches drift. `main` is the only branch that matches Railway. If the operator's brief names a different branch, **fetch `origin/main` anyway and check whether your target branch is behind**. If it is, push back and confirm.

### 3. One PR per session, against `main`, never push directly

Even one-line fixes go through a PR. The PR body must include:

- What changed (short summary table)
- Acceptance checklist with evidence (`npm run typecheck`, `npm test`, `npm run lint` output)
- Railway implications, if any
- Cross-repo CRASH/DECISION log links

### 4. Update the four governance files at end of session

For every session that touches `zerotrading-core` (or any deployed repo):

| File | When to update |
|---|---|
| `ai/handoffs/CURRENT-STATE.md` | Always — bump `Latest commit:`, refresh "Do Not Regress" |
| `ai/summaries/DECISION-LOG.md` | If any architectural choice was made |
| `ai/summaries/CRASH-AND-FIX-LOG.md` | If any bug was found or fixed |
| `ai/summaries/YYYY-MM-DD-HHMM-<name>.md` | Always — your handoff to the next agent |

### 5. Verify before declaring "fixed"

"Pushed" is not the same as "fixed." For Railway deploys, hit `/api/status` or `/health` on the live URL and read the response. For code fixes, confirm the symptom is gone from live data, not just from your own diff.

### 6. Never alter the protected core without explicit instruction

The execution state machine, accounting/ledger, and reconciliation paths are protected. If your task touches:

- `src/machine/`
- `src/engine/effects.ts` (accounting effects)
- `src/engine/recovery.ts`
- Strategy thresholds, regime weights, sizing config

...stop and ask the operator before changing semantics. Adding a new event, new effect, or new field is usually fine. Changing how an existing one resolves is not.

### 7. Fail closed on ambiguity

If you are unsure whether a change is safe, do not ship it. Document the ambiguity in the PR or summary and ask the operator. A delayed answer is cheaper than a regression on a live trading bot.

---

## Repository map

You'll be told which repo to work in. Here's the relationship:

| Repo | Role | Branch protection target |
|---|---|---|
| `zerotrading` (parent) | Governance, docs, prompts, research, decision logs | `main` |
| `zerotrading-core` | KXBTCD hourly bot — **LIVE on Railway** | `main` (deploys) |
| `zerotrading-15m` | KXBTC15M 15-minute bot (Phase 1 skeleton) | `main` |
| `trader-retro`, `kalshi-15m-bot`, `zerotradingx15minbtc` | Sibling research repos. Do NOT modify unless explicitly asked. | n/a |

**The parent repo (`zerotrading`) is the source of truth for governance.** Code repos read from it. When in doubt, the parent wins.

---

## When you're done

Before you sign off, you must be able to answer "yes" to every one of these:

- [ ] `npm run typecheck` clean
- [ ] `npm test` — all passing
- [ ] `npm run lint` — 0 errors (warnings OK if pre-existing and called out)
- [ ] PR opened against `main` with full body
- [ ] PR merged (or operator explicitly asked you to leave it open)
- [ ] Railway live URL hit, response sanity-checked
- [ ] `CURRENT-STATE.md` updated and pushed to parent repo
- [ ] New dated summary in `ai/summaries/`
- [ ] DECISION-LOG and CRASH-AND-FIX-LOG appended if applicable

If any of these is "no", you are not done. Say so to the operator and explain what's blocking.
