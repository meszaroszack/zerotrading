# Authoritative Design PDFs

## Guidelines

This folder holds the **canonical, authoritative design references** for ZeroTrading Core. These PDFs are protected foundations — they describe the execution state machine, the accounting/fee-correction model, and the overall core architecture.

Agents and humans working on ZeroTrading must treat these PDFs as the source of truth for the core shell. If code, docs, or strategies diverge from these PDFs, the divergence must be either reconciled in code or formally captured with an updated PDF and a `DECISION-LOG.md` entry.

## Expected files

| Filename | Description |
|---|---|
| `zerotrading-core-architecture.pdf` | Overall core architecture: layers, modules, boundaries, ownership |
| `zerotrading-core-execution-statemachine.pdf` | Execution lifecycle FSM: IDLE -> ENTERING -> OPEN -> EXITING -> SETTLED + recovery/shutdown |
| `zerotrading-core-accounting-fee-corrections.pdf` | P&L truth model, fee handling, reconciliation, audit corrections |

If any of these files are missing, the repo is considered **incomplete**. Restore them from prior session attachments before continuing serious work.

## How to add or update them

These are binary files and cannot be created through the GitHub web editor by typing text. Add them via one of these paths:

1. **Drag-and-drop** in GitHub web UI: navigate to `docs/pdfs/`, click "Add file" -> "Upload files", drop the PDF in, commit.
2. **Local git**:
   ```
   git checkout -b chore/docs-pdfs
   cp /path/to/zerotrading-core-architecture.pdf docs/pdfs/
   git add docs/pdfs/zerotrading-core-architecture.pdf
   git commit -m "docs(pdfs): add core architecture PDF"
   git push origin chore/docs-pdfs
   ```
3. **gh CLI** via a script that uploads all three at once.

When updating an existing PDF, bump a version note in `ai/summaries/DECISION-LOG.md` describing what changed and why.

## What does NOT belong here

- Marketing or external-facing PDFs (those go in `docs/external/` if needed)
- Screenshots or PNGs (use `docs/diagrams/` or `docs/screenshots/`)
- Anything containing secrets, account IDs, or live infrastructure details
- Old or experimental design drafts (keep those in a separate `docs/drafts/` folder)

## Cross-references

These PDFs are referenced from:

- `README.md`
- `docs/ARCHITECTURE-ZEROTRADING-CORE.md`
- `ai/system/MODEL-SPEC.md`
- `ai/system/REPO-PRINCIPLES.md`
- `ai/prompts/BUILD-LAUNCH-BOT.md`

If you rename or relocate any of these PDFs, update every reference above in the same PR.
