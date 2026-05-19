# Master Cross-Agent Prompt

This is the canonical prompt used to initialize any AI agent (Perplexity, Cursor, Claude, GPT-4, or any other) working in this repository. Paste this at the start of every new session.

---

```
You are working in the ZeroTrading repository.
GitHub: https://github.com/meszaroszack/zerotrading
Live app: https://zerotrading-core-production.up.railway.app

ZeroTrading is an AI-first, open-knowledge prediction market platform built on Kalshi.
Deployment: Railway. Persistence and analytics: Supabase.

## STEP 1: READ THESE FILES FIRST (in order)

1. README.md
2. ai/system/README-AI-SYSTEM.md
3. ai/system/MODEL-SPEC.md
4. ai/system/OPEN-OPSEC-POLICY.md
5. ai/system/REPO-PRINCIPLES.md
6. docs/ARCHITECTURE-ZEROTRADING-CORE.md
7. ai/handoffs/CURRENT-STATE.md (if it exists)
8. ai/summaries/DECISION-LOG.md

Do not proceed until you have read all of the above.

## STEP 2: PROTECTED FOUNDATIONS

Do NOT bypass or casually alter:
- The execution state machine (see docs/pdfs/zerotrading-core-execution-statemachine.pdf)
- The accounting and fee correction logic (see docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf)
- The architecture boundaries (see docs/pdfs/zerotrading-core-architecture.pdf)

Do NOT commit secrets, credentials, API keys, or live account identifiers.

## STEP 3: CURRENT WORK PRIORITIES

1. Mystic-style KXBTC15M late-window strategy module integrated into ZeroTrading Core.
2. BTC hourly models - stabilize accounting and arithmetic, reuse existing work.
3. Continuously mine and adapt viable models from GitHub, sports, and weather markets.
4. 24/7 market monitoring layer: always-on snapshots, anomaly detection, bubble-map analytics.
5. All AI passes, bugs, fixes, and findings must be committed to GitHub - not left in chat.

## STEP 4: SESSION OUTPUT CONTRACT

At the end of every session you MUST:
- Update or create relevant docs under docs/ or ai/
- Append an entry to ai/summaries/DECISION-LOG.md
- Create a fresh-session summary (see ai/handoffs/FRESH-SESSION-TEMPLATE.md)
  stored in ai/summaries/ with a timestamped filename: YYYY-MM-DD-HH-summary.md
- Push all changes to GitHub. Chat-only output is not acceptable.
- Satisfy ai/checklists/SESSION-OUTPUT-CONTRACT.md

## STEP 5: STYLE AND SAFETY

- Small, reviewable commits. Conventional commit messages.
- Fail closed: if safety or accounting is ambiguous, do NOT expand live behavior.
- Explainability is a feature: every trade and every skip must be legible to an operator.
- Keep the repo understandable to both humans and AI agents at all times.

## CURRENT TASK

[REPLACE THIS WITH ONE CLEAR TASK DESCRIPTION]
Example: "Wire a Mystic-style KXBTC15M strategy module into ZeroTrading Core in paper mode only,
and document the design in docs/STRATEGY-KXBTC15M.md."

## REQUIRED RESPONSE FORMAT

1. Objective (what you are doing and why)
2. Files you will read
3. Files you will create or modify
4. Step-by-step plan
5. Safety and risk notes
6. Changes for DECISION-LOG
7. Fresh-session summary block
```
