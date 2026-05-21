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

Current task for this run:
[DESCRIBE ONE CLEAR TASK HERE]
```
