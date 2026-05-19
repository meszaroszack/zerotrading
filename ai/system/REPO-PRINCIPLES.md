# Repository Principles

These principles govern how ZeroTrading is built, documented, and evolved.

## 1. Documentation and Approval First

Design intent is documented before major implementation work begins. AI-generated changes must be explainable and traceable. No significant refactor without a corresponding doc update.

## 2. Single Source of Truth

ZeroTrading Core (execution state machine + accounting + reconciliation) is the authoritative shell. Strategy modules plug into the shell — they do not replace it. The PDFs under `docs/pdfs/` are the source-of-truth design references for the core.

## 3. Open and Reproducible

No hidden algorithm files. No README theater. Docs must match reality. If something is broken or incomplete, say so in the docs.

## 4. AI Agents as First-Class Contributors

AI agents are expected to push documentation, code, and research findings directly to GitHub. A session that produces only chat output without a GitHub commit is incomplete.

## 5. Continuous Research Log

Every bug, fix, anomaly, strategy change, or market observation is logged. The monitoring and anomaly layer is treated as a product, not an afterthought.

## 6. Railway + Supabase as Deployment Standard

All production deployments use Railway. All persistence and analytics use Supabase. Configuration lives in environment variables, not in code.

## 7. Wikipedia Model

This platform should eventually be the Wikipedia of prediction market research: open, inspectable, improvable by the community, with no important files missing.
