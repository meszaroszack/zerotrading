# Fresh Session Handoff Template

## Guidelines

This template is the **drift catcher** for ZeroTrading. Every serious AI session (Perplexity, Cursor, Claude, Comet, etc.) must produce a filled copy at the end of its work and commit it to `ai/summaries/` with a timestamped filename like `YYYY-MM-DD-HHMM-<short-topic>.md`.

Purpose:
- New sessions can resume work without re-reading the entire chat history.
- Cross-agent handoffs stay consistent.
- Decisions, blockers, and next steps are durable.

Do not delete or shorten sections. If a section does not apply, write `n/a` and explain briefly.

---

# FRESH SESSION SUMMARY

## Project
ZeroTrading - AI-first, open-knowledge prediction market platform built on Kalshi.

Repo: https://github.com/meszaroszack/zerotrading
Live app (current): https://zerotrading-core-production.up.railway.app
Deployment: Railway (app) + Supabase (persistence/analytics)

## What ZeroTrading is
A prediction-market operating platform with two tracks:
1. Live KXBTC15M execution engine (Mystic-style late-window strategy as flagship)
2. 24/7 market intelligence and anomaly archive

## Protected foundations (do not modify casually)
- Execution state machine (`docs/pdfs/zerotrading-core-execution-statemachine.pdf`)
- Accounting / fee corrections (`docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf`)
- Core architecture boundaries (`docs/pdfs/zerotrading-core-architecture.pdf`)

## Current priorities (in order)
1. Mystic-style KXBTC15M 15-minute launch strategy
2. Harden BTC hourly models (fix accounting arithmetic)
3. Mine and adapt viable models from GitHub, sports, weather, prior research
4. 24/7 monitoring layer: market archive, anomaly detection, bubble maps, analytics graphics
5. Every step, bug, fix, and AI pass produces a doc update + DECISION-LOG entry

## Non-negotiables
- Exchange truth wins over local assumptions
- Restart-safe state restoration
- No duplicate entries
- No realized P&L before fill confirmation
- Visible skip reasons + candidate scores
- Visible reconciliation/correction events
- Fail closed on ambiguity
- No secrets, keys, or account identifiers in the repo

---

## SESSION-SPECIFIC SECTION (fill this in)

### Session metadata
- Date / time (ET):
- Agent / model used:
- Operator (human):
- Branch:
- Mode (paper / safe-live / live / docs-only):

### Objective
_What was this session trying to accomplish in one or two sentences?_

### Checklist (status)
- [ ] Read README + ai/system docs
- [ ] Reviewed authoritative PDFs in docs/pdfs/
- [ ] Reviewed latest DECISION-LOG entries
- [ ] Made changes scoped to the objective
- [ ] Updated relevant docs
- [ ] Added/updated tests or evals
- [ ] Appended a DECISION-LOG entry
- [ ] Committed this fresh-session summary to ai/summaries/

### Findings / changes
_Files inspected, files modified, behavior changed, bugs fixed, decisions made._

### Risks / blockers
_Anything ambiguous, unsafe, unfinished, or requiring human approval before going further._

### Exact next prompt
_The literal copy-paste prompt the next agent should run to continue this work._

### Open questions
_Things this session could not resolve and needs operator input on._

---

## Filename convention when committing
```
ai/summaries/2026-05-19-1700-kxbtc15m-strategy-scaffold.md
ai/summaries/2026-05-19-2230-accounting-recon-fix.md
```

## Reminder
If you ended a session without producing this artifact, the session is considered **incomplete** regardless of code changes.
