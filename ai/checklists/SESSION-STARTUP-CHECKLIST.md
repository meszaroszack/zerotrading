# Session Startup Checklist

Run this checklist at the start of every session, before writing any code. It takes under a minute and prevents the 2026-05-23-class divergence incident (see `ai/guardrails/BRANCH-AND-MERGE-DISCIPLINE.md`).

---

## Copy-paste block

```bash
# 1. Sync everything
git fetch --all --prune

# 2. Show recent history on main (READ THIS — don't skip)
git log origin/main -10 --oneline

# 3. Show your current divergence from main
git status
git log --oneline --left-right HEAD...origin/main | head -20

# 4. List open PRs (if any conflict with your task, stop and read them first)
gh pr list --state open

# 5. Show the latest dated summary
ls -t ai/summaries/2*.md | head -1 | xargs head -30
```

In the parent repo (`zerotrading`), also:

```bash
# 6. Read the current state pointer
cat ai/handoffs/CURRENT-STATE.md | head -60

# 7. Confirm Latest commit: line matches what step 2 showed
```

---

## The four questions you must be able to answer

After running the block above, state these explicitly in your session output:

1. **Where am I starting from?** Example: `Branched fix/x off origin/main @ 7ea6400 on 2026-05-23 04:50 ET.`
2. **What's the most recent operator-visible state?** Example: `Last session ended at commit f4b7bdc with bot live on Railway, balance ~$26.42, 2 wins 1 loss.`
3. **What "Do Not Regress" items apply to my task?** Pull them from `CURRENT-STATE.md`.
4. **Are there open PRs that touch the files I'll touch?** If yes, what do I do about that?

If you can't answer any one of these, you haven't done the checklist. Do it.

---

## Anti-patterns to recognize in your own behavior

- "The operator told me to branch off `core-engine-review-v1`, so I will." — **No.** Fetch `origin/main` and check whether `core-engine-review-v1` is behind. If it is, surface that before proceeding.
- "The brief is clear, I don't need to read CURRENT-STATE." — **No.** Briefs are written from the operator's memory. CURRENT-STATE is written from the last agent's measurement. They drift. Read CURRENT-STATE.
- "There are no open PRs, so I'm safe." — **Not necessarily.** Check recently-merged PRs too: `gh pr list --state merged --limit 5`. If main moved in the last 24 hours, something is in flight.
- "I'll do the docs at the end." — **You won't.** The end-of-session token budget is always tighter than the start-of-session budget. Open the doc files in your context now so they get updated automatically as the session proceeds.

---

## The 60-second rule

If reading all of these files would take longer than 60 seconds, you're doing it wrong. The checklist exists because the *cost* of skipping is hours, and the *cost* of doing it is under a minute. Pay the minute. Always.
