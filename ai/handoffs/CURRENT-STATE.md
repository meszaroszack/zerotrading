# CURRENT STATE

_Live pointer file. Overwrite at end of every session. Append-only history lives in `ai/summaries/DECISION-LOG.md`._

**Last updated:** 2026-05-19 23:00 ET
**Updated by:** Comet (browser agent) on behalf of meszaroszack

## Where we are
- AI-first repo scaffold complete: `ai/`, `docs/`, `ops/runbooks/`, `research/logs/`, `.cursor/rules/`.
- Authoritative PDFs present in `docs/pdfs/` (architecture, execution FSM, accounting/fee-corrections).
- MASTER-PROMPT v2 active at `ai/prompts/MASTER-PROMPT.md` (research capture, push-honesty, mode discipline).
- Strategy scaffolds in place: `docs/STRATEGY-KXBTC15M.md`, `STRATEGY-BTC-HOURLY.md`, `STRATEGY-Weather.md`.
- No code yet. No live deploy on this repo. Live beta app (`zerotrading-core-production.up.railway.app`) is a different GitHub - treat as read-only reference.

## What is in flight
- Reviewing user's prior 15-minute market builds (URLs/paths TBD by user).
- Each prior build will get a `research/adapted/<source>-15m.md` summary + DECISION-LOG entry (Status: proposed).
- Output: a v2 of `docs/STRATEGY-KXBTC15M.md` synthesizing reusable patterns.

## Next concrete step
1. User points to prior 15-min builds (GitHub URLs, local paths, or pasted code).
2. Agent reads each, writes one `research/adapted/<source>-15m.md`.
3. Agent appends DECISION-LOG (Status: proposed) per build.
4. Agent updates `docs/STRATEGY-KXBTC15M.md` v2 with synthesis.
5. Agent overwrites this file and writes `ai/summaries/2026-05-19-23-summary.md`.

## Open questions / blockers
- Which prior 15-min builds to start with (user input pending).
- Code language for KXBTC15M module (Python vs TS - pending user call).
- Whether to mirror live-beta-app code into this repo or keep clean rebuild (pending port-audit decision).

## Do not regress
- Execution FSM (see `docs/pdfs/zerotrading-core-execution-statemachine.pdf`).
- Accounting / fee corrections (see `docs/pdfs/zerotrading-core-accounting-fee-corrections.pdf`).
- OPSEC: no secrets, no account IDs in repo.
- Mode discipline: paper -> safe-live -> live.
