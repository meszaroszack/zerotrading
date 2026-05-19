# Session Output Contract

## Guidelines

This checklist defines what every AI session in the ZeroTrading repo must produce before it is considered complete. It applies to all agents (Perplexity, Cursor, Claude, Comet, custom scripts) and to human operators driving them.

A session that changes code or docs but does not satisfy this contract is considered **incomplete** and must be re-opened.

## Hard requirements

### Pre-work
- [ ] Read `README.md`
- [ ] Read `ai/system/MODEL-SPEC.md`
- [ ] Read `ai/system/OPEN-OPSEC-POLICY.md`
- [ ] Read `ai/system/REPO-PRINCIPLES.md`
- [ ] Check `docs/pdfs/` for authoritative design references
- [ ] Check most recent `ai/summaries/*.md` entries
- [ ] Check `ai/summaries/DECISION-LOG.md` for recent decisions

### Scope discipline
- [ ] Objective for this session is stated in one or two sentences
- [ ] Changes are scoped to that objective
- [ ] No casual changes to the execution state machine, accounting, or core architecture
- [ ] No commits of secrets, credentials, or account-specific identifiers

### Output artifacts
- [ ] Code changes (if any) are reviewable and small
- [ ] Docs updated where behavior or design changed
- [ ] Tests or evals updated where logic changed
- [ ] `ai/summaries/DECISION-LOG.md` has an appended entry for any meaningful decision or fix
- [ ] A filled `ai/handoffs/FRESH-SESSION-TEMPLATE.md` copy is committed to `ai/summaries/` with a timestamped filename

### Safety
- [ ] If anything is ambiguous regarding live trading, the session failed closed (no expansion of live behavior)
- [ ] If accounting or reconciliation logic was touched, the change is documented and clearly labeled
- [ ] If new external data sources or endpoints were introduced, they are documented and risk-noted

### Communication
- [ ] Final response to operator includes:
  - Objective
  - Files inspected
  - Files modified / created
  - Decisions made
  - Risks / blockers
  - Exact next prompt for the next session

## Failure modes that void this contract

- Claiming "done" without listing changed files
- Adding strategy logic without updating docs or evals
- Adding live execution paths without paper/safe-live equivalents
- Editing accounting logic without a DECISION-LOG entry
- Committing PDFs or images without updating the relevant README index
- Leaving the repo in a state that requires reading chat history to understand

## How to use this file

At the start of a session, copy the checklist into the session scratch space.
At the end of a session, verify every box and commit the filled handoff summary.
If you cannot satisfy a box, document why in the handoff summary's `Risks / blockers` section.
