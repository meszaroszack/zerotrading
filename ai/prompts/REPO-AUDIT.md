# Prompt: Repository Audit

Use this prompt when you want an AI agent to audit the repo for drift, missing docs, or stale content.

---

```
You are auditing the ZeroTrading GitHub repository for quality and completeness.
Repo: https://github.com/meszaroszack/zerotrading

AUDIT CHECKLIST:
- [ ] Every strategy module has a corresponding doc in docs/
- [ ] ai/summaries/DECISION-LOG.md is up to date
- [ ] ai/handoffs/CURRENT-STATE.md reflects current reality
- [ ] docs/ARCHITECTURE-ZEROTRADING-CORE.md is accurate
- [ ] No secrets or credentials are present anywhere
- [ ] CHANGELOG.md is current
- [ ] All ai/prompts/ files match current priorities
- [ ] Eval cases exist for active strategies
- [ ] Runbooks cover all deployment and recovery scenarios
- [ ] README.md matches current architecture

OUTPUT:
- Pass/fail for each checklist item
- List of files that need updating with specific gaps
- PRs or direct commits to fix gaps
- DECISION-LOG entry
- Fresh-session summary pushed to GitHub
```
