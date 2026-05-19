# Contributing to ZeroTrading

ZeroTrading welcomes contributions from humans and AI agents equally, provided they follow these rules.

## For All Contributors

1. Read `README.md` and `ai/system/README-AI-SYSTEM.md` before making any changes.
2. Respect the protected execution, accounting, and reconciliation core. Do not alter these without explicit approval.
3. Document your work. Every non-trivial change should update or create a doc in `docs/` or `ai/`.
4. Follow the Session Output Contract in `ai/checklists/SESSION-OUTPUT-CONTRACT.md`.
5. Do not commit secrets, credentials, API keys, or live account identifiers under any circumstances.

## For AI Agents

- You must update `ai/summaries/DECISION-LOG.md` with a short entry after every significant pass.
- You must produce a fresh-session summary using `ai/handoffs/FRESH-SESSION-TEMPLATE.md` and store it in `ai/summaries/` with a timestamped filename.
- You must reference the master prompt at `ai/prompts/MASTER-PROMPT.md` and confirm you have read it.
- If you are unsure whether a change is safe, fail closed and document the ambiguity.

## PR Guidelines

- Keep PRs small and reviewable.
- Write clear commit messages that describe what changed and why.
- Reference any relevant docs, PDFs, or prior decisions.
- Add tests or eval cases where feasible.
