# Master Cross-Agent Prompt (v2)

This is the canonical prompt used to initialize any AI agent (Perplexity, Cursor, Claude, GPT, or any other) working in this repository. Paste this at the start of every new session.

---

```
MASTER CONTEXT PROMPT - ZEROTRADING

GitHub: https://github.com/meszaroszack/zerotrading
Live Beta App: https://zerotrading-core-production.up.railway.app  (read-only reference; different GitHub; do NOT assume code parity with this repo until a port-audit DECISION-LOG entry exists)
Deployment: Railway (app) + Supabase (persistence/analytics)

You are working in the ZeroTrading repository. Before doing anything, fetch and read these files from https://github.com/meszaroszack/zerotrading :

1. README.md
2. ai/system/README-AI-SYSTEM.md
3. ai/system/MODEL-SPEC.md
4. ai/system/OPEN-OPSEC-POLICY.md
5. ai/system/REPO-PRINCIPLES.md
6. docs/ARCHITECTURE-ZEROTRADING-CORE.md
7. ai/handoffs/CURRENT-STATE.md   (always points to latest state)
8. ai/summaries/DECISION-LOG.md   (append-only history)
9. Latest dated summary in ai/summaries/YYYY-MM-DD-HH-summary.md (highest filename wins)
10. ai/guardrails/PERSISTENCE-RECONCILIATION.md  (REQUIRED — blocks repeat of fatal flaw from all prior builds)
11. ai/summaries/CRASH-AND-FIX-LOG.md             (REQUIRED before any code — every known crash and fix)
12. ops/runbooks/RAILWAY-KNOWN-ISSUES.md          (read before any Railway deploy or requirements.txt change)
    → file lives in zerotradingx15minbtc beta repo; canonical Railway issue list for the whole platform

Authoritative design PDFs (in docs/pdfs/):
- zerotrading-core-execution-statemachine.pdf
- zerotrading-core-accounting-fee-corrections.pdf
- zerotrading-core-architecture.pdf

Work priorities (in order):
1. Mystic-style KXBTC15M 15-minute strategy module (flagship launch strategy)
2. BTC hourly models - harden existing, fix accounting arithmetic
3. Mine and adapt viable models from GitHub, sports, weather, and prior research
4. 24/7 monitoring layer - market archive, anomaly detection, bubble maps, analytics/data-science graphics

Modes: paper -> safe-live -> live. Never skip safe-live. New strategies start in paper.

Non-negotiables:
- All bugs, fixes, and AI passes produce: doc update + DECISION-LOG entry + CURRENT-STATE refresh
- Do NOT alter execution state machine or accounting core without explicit instructions
- Do NOT commit secrets, keys, or account identifiers
- Exchange truth wins over local assumptions
- Fail closed on ambiguity
    - ALL state MUST be persisted in Supabase (NEVER in-memory or file-based). See ai/guardrails/PERSISTENCE-RECONCILIATION.md
    - Every boot MUST reconcile positions and balance with Kalshi exchange before trading
    - P&L MUST come from actual Kalshi fill prices and settlements, not from estimates
- Cite file paths with line refs when referencing repo content (e.g., docs/STRATEGY-KXBTC15M.md:42)
- If you cannot push directly, output a copy-paste git/gh command block and STOP - do not pretend a push occurred
- Any bug found or fixed MUST be logged in ai/summaries/CRASH-AND-FIX-LOG.md before the session ends

VERIFY BEFORE CLOSING RULE:
After writing any fix, you MUST verify it worked before telling the operator it is done.
For Railway deploys: hit /api/status or /health on the live URL and read the response.
For code fixes: confirm the symptom is gone from live data, not just from reading your own diff.
"Pushed" is not the same as "fixed." Do not declare a fix complete until you have observed
the corrected behavior in the running system. If you cannot verify (no URL, no access),
say so explicitly — do not imply it worked.

DIAGNOSE BEFORE ASKING:
When the operator reports a crash, read the source files and traceback FIRST. Diagnose from code.
The operator's credentials, copy-paste, and configuration are correct until the code proves otherwise.
Never ask the operator to re-verify something you have not personally inspected.

RAILWAY DEPLOY RULE:
Before diagnosing any Railway crash, check ops/runbooks/RAILWAY-KNOWN-ISSUES.md (in beta repo).
Every recurring deploy failure is documented there. Apply the fix silently. If it is a new failure,
diagnose from the traceback, fix it, then add it to that file.

PAST CONTEXT RULE:
This operator has extensive prior session history. Load past context before starting work.
GitHub is the written memory — read it. Prior sessions are the institutional memory — search them.
Never make the operator re-explain something already documented.

Research & idea capture (every interaction):
- Raw notes, transcripts, screenshots, external research:
    research/notes/YYYY-MM-DD-<slug>.md
- Adapted/distilled patterns from other repos or strategies:
    research/adapted/<source>-<topic>.md   (cite source URL + commit hash if applicable)
- Mystic-specific findings (the original system being ported):
    research/mystic/<topic>.md
- Cross-system synthesis (how Mystic + BTC hourly + weather + sports interact):
    docs/SYSTEM-SYNTHESIS.md   (append-only sections per session)
- Any new direction or pivot idea -> DECISION-LOG entry with Status: proposed

Session output contract (end of every session you MUST):
1. Update or create relevant docs under docs/ or ai/
2. Append an entry to ai/summaries/DECISION-LOG.md
3. Overwrite ai/handoffs/CURRENT-STATE.md with the new live state
4. Create a fresh-session summary at ai/summaries/YYYY-MM-DD-HH-summary.md
   (shape per ai/handoffs/FRESH-SESSION-TEMPLATE.md and ai/schemas/session-summary.schema.json)
5. If a strategy or architecture decision changed, update the corresponding
   docs/STRATEGY-*.md or docs/ARCHITECTURE-ZEROTRADING-CORE.md in the same commit batch.
6. If any bug was found or fixed: append to ai/summaries/CRASH-AND-FIX-LOG.md with commit hash.
   Also update beta repo (meszaroszack/zerotradingx15minbtc) crash log if relevant.

Current task for this run:
[DESCRIBE ONE CLEAR TASK HERE]
```
