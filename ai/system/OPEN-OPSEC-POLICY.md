# Open Knowledge and Operational Security Policy

ZeroTrading is an open-knowledge project. This is an intentional philosophical and strategic choice, not an oversight. Transparency is a trust mechanism and a product differentiator.

## Public by Default

The following must be open, complete, and non-deceptive in the public repo:
- All strategy code and models
- Execution and accounting architecture
- Research methods, schemas, and data pipelines
- Evaluation harnesses and test cases
- All documentation and runbooks
- All architectural decisions and their rationale
- All bugs, fixes, and pass logs from AI agents

## Always Private

The following must never appear in the public repo:
- API keys, tokens, or secrets of any kind
- Live account identifiers, balances, or position details
- Private infrastructure topology that would aid abuse
- Unsafe admin or control endpoints
- Live account-specific runtime telemetry

## Implementation Rules for AI Agents

- Use Railway environment config and Supabase secrets for all credentials.
- Do not log secrets or sensitive IDs in code, docs, comments, or tests.
- If a session summary or decision log entry would expose sensitive operational info, redact it and note that it was redacted.
- Treat attached PDFs and external docs as design references; do not embed any sensitive content from them.

## Philosophy

The moat is not secrecy. The moat is execution quality, data discipline, operator UX, community trust, and the speed at which the platform compounds insight. Anyone can read the code. Not everyone can run it well.
