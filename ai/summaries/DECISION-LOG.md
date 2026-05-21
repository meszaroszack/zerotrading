# Decision Log

## Guidelines

This is the **append-only decision log** for ZeroTrading. Every meaningful design choice, fix, pivot, accounting correction, or strategy change gets a new entry here.

Why append-only:
- Decisions over time form the institutional memory of the project.
- Future agents and operators need to understand *why* things are the way they are.
- Reversing a prior decision means appending a new entry that explicitly supersedes it.

## Entry format

Each entry should follow this shape:

```
## YYYY-MM-DD HH:MM ET - <short title>

**Author:** <human or agent name>
**Session:** <link or filename of session summary in ai/summaries/>
**Status:** decided | proposed | superseded by <date>
**Scope:** strategy | execution | accounting | architecture | docs | ops | research

### Context
What triggered this decision. What was unclear, broken, or being debated.

### Decision
What was decided, in plain language. One sentence if possible.

### Rationale
Why this choice over alternatives.

### Consequences
What changes downstream because of this. Risks. Open follow-ups.

### Related files
- <path/to/file1>
- <path/to/file2>
```

## Rules

1. **Never delete entries.** Supersede them by appending a new entry that references the old one.
2. **Date entries in ET** to match the operator's primary timezone.
3. **Link to the session summary** in `ai/summaries/` that produced the decision.
4. **Keep entries short.** Long context belongs in the session summary; the log is the index.
5. **Tag scope** so we can filter later (strategy, execution, accounting, architecture, docs, ops, research).

---

## 2026-05-19 17:00 ET - Initialize AI-first repo scaffold

**Author:** Comet (Perplexity) + operator
**Session:** Initial scaffold session (multi-turn)
**Status:** decided
**Scope:** docs, architecture

### Context
Work across multiple sessions and agents had been losing context between turns. The repo needed an AI-first structure with prompts, handoffs, checklists, evals, schemas, and authoritative design references separated from product code.

### Decision
Adopt an AI-first repo layout with:
- `ai/system/` for canonical agent specs
- `ai/prompts/` for reusable master prompts
- `ai/handoffs/` for session templates
- `ai/checklists/` for non-negotiable contracts
- `ai/summaries/` for session outputs and this decision log
- `ai/evals/` and `ai/schemas/` for measurable quality
- `docs/pdfs/` for authoritative design PDFs
- `docs/diagrams/` for visual architecture artifacts
- `ops/runbooks/` for traditional ops docs
- `research/logs/` for monitoring archive
- `.cursor/rules/` for Cursor-specific agent rules

### Rationale
Dual goals: (1) keep AI agents aligned across sessions and tools, (2) preserve traditional SaaS hygiene. Mirrors patterns from prompt-engineering and Cursor rule guidance.

### Consequences
- All future sessions must end with a fresh-session summary in `ai/summaries/` and a decision log entry when appropriate.
- Strategy and code modules should reference protected PDFs as design source of truth.
- The repo is intentionally documentation-first; code lands behind docs.

### Related files
- `README.md`
- `ai/system/MODEL-SPEC.md`
- `ai/system/OPEN-OPSEC-POLICY.md`
- `ai/system/REPO-PRINCIPLES.md`
- `ai/handoffs/FRESH-SESSION-TEMPLATE.md`
- `ai/checklists/SESSION-OUTPUT-CONTRACT.md`
- `docs/diagrams/README.md`
- `docs/pdfs/README.md`

---

## 2026-05-19 17:00 ET - Open-knowledge + OpSec posture

**Author:** Comet (Perplexity) + operator
**Session:** Initial scaffold session
**Status:** decided
**Scope:** docs, ops

### Context
Operator wants ZeroTrading to be genuinely open like Wikipedia (no fake openness, no missing core files, no hidden algorithms) while still maintaining strong operational security around secrets, infrastructure, and exploit-enabling details.

### Decision
Adopt a hybrid posture:
- **OPEN:** code, algorithms, architecture, research methods, docs, schemas
- **PRIVATE:** secrets, credentials, account-specific runtime data, exploit-enabling live operational details

Moat comes from execution quality, data discipline, operator UX, and research compounding - not from hiding important parts.

### Rationale
Matches operator's ethical stance, differentiates from "published recipe / private code" patterns like Mystic, and is compatible with open-source security best practices.

### Consequences
- README and AI system docs must clearly state this posture.
- SECURITY.md governs threat model and secret handling.
- No README theater - if a feature is described, the code should exist.

### Related files
- `README.md`
- `SECURITY.md`
- `ai/system/OPEN-OPSEC-POLICY.md`

---

## 2026-05-19 17:00 ET - Launch wedge = Mystic-style KXBTC15M

**Author:** Comet (Perplexity) + operator
**Session:** Initial scaffold session
**Status:** decided
**Scope:** strategy

### Context
Multiple candidate strategies exist (BTC hourly, weather, arbitrage, sports). Operator needs one focused launch wedge to demonstrate viability and generate revenue while broader platform builds.

### Decision
Flagship launch strategy is a Mystic-inspired KXBTC15M late-window sniper running on the ZeroTrading Core shell. BTC hourly models remain as secondary references. Weather and arb are roadmap items, not launch items.

### Rationale
- Mystic recipe is the most concrete published-claim strategy with credible documented results.
- KXBTC15M is fast, demo-friendly, and emotionally legible.
- Existing core shell already supports execution lifecycle and accounting needed.

### Consequences
- All launch-track engineering effort prioritizes this strategy.
- A `docs/STRATEGY-KXBTC15M.md` should be created to capture the exact recipe and parameters.
- Risk controls (entry window, RSI band, volatility cap, balance tiers, cooldown) must be encoded.

### Related files
- `ai/prompts/BUILD-LAUNCH-BOT.md`
- `docs/STRATEGY-KXBTC15M.md` (to be created)


## 2026-05-19 23:00 ET - AI-first scaffold completion + PDFs uploaded

**Author:** Comet (browser agent) on behalf of meszaroszack
**Session:** ai/summaries/2026-05-19-23-summary.md (pending)
**Status:** decided
**Scope:** docs | architecture | governance

### Context
Repo needed the remaining AI-first scaffolds (research logs, cursor rules, architecture index, strategy specs) and the three authoritative PDFs in `docs/pdfs/`.

### Decision
Committed 7 scaffold files directly to `main`:
- `research/logs/README.md`
- `.cursor/rules/core-product.mdc`
- `.cursor/rules/trading-safety.mdc`
- `docs/ARCHITECTURE-ZEROTRADING-CORE.md`
- `docs/STRATEGY-KXBTC15M.md`
- `docs/STRATEGY-BTC-HOURLY.md`
- `docs/STRATEGY-Weather.md`

User uploaded the three authoritative PDFs to `docs/pdfs/`:
- `zerotrading-core-architecture.pdf`
- `zerotrading-core-execution-statemachine.pdf`
- `zerotrading-core-accounting-fee-corrections.pdf`

References in `docs/ARCHITECTURE-ZEROTRADING-CORE.md` now resolve.

### Consequences
- Master prompt (`ai/prompts/MASTER-PROMPT.md`) Step 1 read-list is now fully satisfiable.
- Protected foundations referenced in Step 2 are physically present in the repo.
- Next sessions can cite PDFs as authoritative; deviations require new DECISION-LOG entry.

### Next
- Create `ai/handoffs/CURRENT-STATE.md` so MASTER-PROMPT Step 1 item 7 resolves.
- Begin work on KXBTC15M Mystic-style launch wedge implementation per `docs/STRATEGY-KXBTC15M.md`.
- Stand up 24/7 monitoring writer into `research/logs/` JSONL.


## 2026-05-19 23:30 ET - MASTER-PROMPT v2 + research scaffolds + CURRENT-STATE

**Author:** Comet (browser agent) on behalf of meszaroszack
**Session:** ai/summaries/2026-05-19-23-summary.md (pending)
**Status:** decided
**Scope:** governance | research | docs

### Context
Needed v2 of MASTER-PROMPT to enforce research-capture-per-interaction policy, honest push behavior, mode discipline, and CURRENT-STATE pointer. Also needed physical folder scaffolds for research notes, adapted patterns, and Mystic-specific findings.

### Decision
- Updated `ai/prompts/MASTER-PROMPT.md` to v2 with: research & idea capture block, push-honesty rule, mode discipline, CURRENT-STATE requirement, live-beta-app read-only clarification, file-path citation rule.
- Created `ai/handoffs/CURRENT-STATE.md` as the live state pointer (overwritten each session).
- Created `research/notes/README.md` with note template and naming convention.
- Created `research/adapted/README.md` with adapted-pattern template.
- Created `research/mystic/README.md` with Mystic-finding template.
- Created `docs/SYSTEM-SYNTHESIS.md` as the append-only cross-system synthesis doc with initial seed entry.

### Consequences
- Every future agent session now has a required research-capture workflow (drop notes -> adapted patterns -> mystic findings -> synthesis -> decision log).
- CURRENT-STATE.md is the boot file for any new agent; always reflects latest state.
- SYSTEM-SYNTHESIS.md captures how strategies and systems interact, growing per session.
- Prior session gaps (no research folder structure, no CURRENT-STATE, no synthesis doc) are closed.

### Next
- User will provide prior 15-min market builds for analysis.
- Each build gets a `research/adapted/<source>-15m.md`.
- Synthesis updates `docs/STRATEGY-KXBTC15M.md` v2.


## 2026-05-20 00:00 ET - Prior 15m builds deep-dive: trader-retro + kalshi-15m-bot

**Author:** Comet (browser agent) on behalf of meszaroszack
**Session:** ai/summaries/2026-05-20-00-summary.md (pending)
**Status:** proposed
**Scope:** research | strategy

### Context
User has two prior KXBTC15M bot builds. Need to understand what works, what broke, and what to port into ZeroTrading.

### Decision
Deep-dived both repos via GitHub. Created detailed analysis files:
- `research/adapted/trader-retro-15m.md` - TypeScript swing bot. Monolithic, InMemory, signals on BTC spot not orderbook. Reusable: Kalshi API client, swing detection pattern, pre-calc stop/target.
- `research/adapted/kalshi-15m-bot-15m.md` - Python multi-strategy bot. Properly modular (client/feed/orders/risk/strategies). Reusable: entire architecture, OrderManager, RiskGuard, WS feed, strategy plugin system, settlement-hold.

Verdict: kalshi-15m-bot is the primary donor codebase. Both share fatal flaw: no persistent state, no exchange reconciliation.

### Consequences
- ZeroTrading KXBTC15M should be built on kalshi-15m-bot's modular architecture (Python), with Supabase persistence and boot reconciliation added.
- trader-retro contributes the Kalshi API client typing pattern and swing-detection idea.
- Three critical fixes identified for any port: (1) persistent state, (2) fill verification + fee accounting, (3) exchange reconciliation loop.
- Language decision needed: Python (kalshi-15m-bot) vs TypeScript (trader-retro). Recommend Python given the more mature codebase.

### Next
- User to confirm language choice (Python recommended).
- Decide starting strategy: simple threshold/last90 first, momentum later.
- Begin porting kalshi-15m-bot src/ structure into ZeroTrading repo with persistence layer.
- Review remaining user builds if any.

- ---

## 2026-05-20 00:30 ET - Mystic Bot analysis and strategy v2 upgrade

**Author:** Comet (browser agent) on behalf of meszaroszack
**Session:** Continuation of KXBTC15M strategy development
**Status:** decided
**Scope:** strategy | research | docs

### Context

User surfaced Mystic Bot (KXBTC15M) live-money performance data from April 12-15, 2026: 134W/3L (~97.8% win rate) on real Kalshi fills. This is the most transparent public performance data found for any KXBTC15M automated strategy. Need to assess credibility, extract learnings, and upgrade ZeroTrading's strategy documentation accordingly.

### Decision

1. **Created `research/mystic/mystic-performance-summary.md`** - Full analysis of Mystic claims, inferred edge logic (convergence wedge), credibility assessment, and what ZeroTrading should adopt vs reject.
2. **Upgraded `docs/STRATEGY-KXBTC15M.md` to v2** - Replaced skeleton with full Mystic-informed convergence strategy spec: entry/exit rules, volatility regime classification, risk guardrails, accounting requirements, execution FSM reference, backtesting requirements, and deployment path (paper -> safe-live -> live).
3. **Created `docs/PERFORMANCE-RECORDS.md`** - Auditable performance log with PERF-001 (Mystic data), verification policy, and minimum sample sizes for strategy approval.

### Consequences

- ZeroTrading now has a concrete, parameterized strategy spec (not just a skeleton) to implement.
- Mystic data is properly documented with caveats (4-day sample, no fee-adjusted PnL, selection bias possible).
- Clear verification policy prevents premature go-live based on unverified third-party claims.
- Strategy v2 explicitly requires 200+ paper trades before safe-live, 500+ before full live.
- Convergence-first approach adopted as primary thesis (buy YES on "BTC stays in range").

### Next

- User to confirm language choice (Python recommended) and starting strategy parameters.
- Begin implementing paper-mode bot based on STRATEGY-KXBTC15M.md v2.
- Backtest convergence logic against historical 15m BTC data (minimum 30 days).
- Update CURRENT-STATE.md to reflect new artifacts.


---

## 2026-05-20 01:00 ET - CRITICAL CORRECTION: KXBTC15M is over/under, not range/wedge

**Author:** Comet (browser agent) on behalf of meszaroszack
**Session:** Market structure correction
**Status:** decided - SUPERSEDES prior entry's convergence/range references
**Scope:** strategy | research | docs | guardrails

### Context

User identified a critical error: all AI-generated KXBTC15M analysis incorrectly described the contract as a range/wedge/convergence product. KXBTC15M is a simple binary over/under contract. One strike, two outcomes: above or below. No ranges, bands, wedges, or convergence zones exist. This is a known recurring AI failure mode.

### Decision

1. Upgraded STRATEGY-KXBTC15M.md to v3 with correct over/under model.
2. Rewrote mystic-performance-summary.md v2 with directional bias detection logic.
3. Fixed PERFORMANCE-RECORDS.md to reference directional momentum, not convergence.
4. Created ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md as required reading for all AI agents.

### Consequences

- ALL prior convergence/wedge/range references for KXBTC15M are now marked as errors.
- Guardrail doc must be referenced in MASTER-PROMPT, .cursor/rules, and all handoffs.
- Strategy reframed as directional prediction: which side of strike will BTC finish?
- This demonstrates a core risk of AI-assisted trading: AI confidently builds on wrong market structure assumptions.

### Next


---

## 2026-05-20 01:15 ET - Kalshi market taxonomy: KXBTCD hourly vs KXBTC15M 15-min

**Author:** Comet (browser agent) on behalf of meszaroszack
**Session:** Market structure documentation
**Status:** decided
**Scope:** guardrails | docs | architecture

### Context

User identified that the live Railway deployment (`zerotrading-core-production.up.railway.app`) targets KXBTCD (hourly), which is a DIFFERENT market than KXBTC15M (15-min). KXBTCD has a ladder of strike prices where you choose a threshold level. KXBTC15M is a simple up/down from the current price. The repo needs to clearly distinguish these and prevent any AI agent from confusing them.

### Decision

1. Created `ai/guardrails/KALSHI-MARKET-REFERENCE.md` - Comprehensive reference covering KXBTCD hourly (ladder, threshold selection, strike distance), KXBTC15M 15-min (single strike, over/under), BTC Price Range (true brackets), settlement mechanics (60s CFB RTI average), fee structure, and common AI errors.
2. Updated `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md` - Added cross-reference to the market reference doc.
3. Updated `ai/handoffs/CURRENT-STATE.md` - Now shows both systems (KXBTCD LIVE on Railway, KXBTC15M planned), live bot status ($4.16 balance, 0 cycles, NO_TRADE decisions), and explicit market distinction table.

### Consequences

- Repo now clearly distinguishes the two target markets at every entry point.
- Live KXBTCD bot status is documented (connected, evaluating, mostly NO_TRADE).
- CURRENT-STATE now leads with a mandatory market table so any agent immediately knows which system targets which market.
- Do-not-regress list now includes "KXBTCD and KXBTC15M are different markets."
- Kalshi fee structure documented: maker orders have NO fees, settlement has NO fees. This is a critical strategic advantage.
- Update CURRENT-STATE.md.
- Wire guardrail into MASTER-PROMPT and .cursor/rules.
- User confirms language choice and begins implementation with correct model.

---

## 2026-05-21 ~02:00 ET - Full build inventory deep-dive complete

**Author:** Comet (browser agent) on behalf of meszaroszack
**Session:** Prior build review and research documentation
**Status:** decided
**Scope:** research | strategy | architecture

### Context

User requested deep-dive of all prior Kalshi trading builds before writing the KXBTC15M strategy spec. Previously only trader-retro and kalshi-15m-bot had been reviewed. Needed to catalog the full portfolio to identify what's reusable vs. what's broken.

### Decision

Deep-dived 7 builds total (5 new this session). Created research/adapted/ files for each:
1. `kalshi-ai-trader-v2-15m.md` - AI-driven regime detection, penny hunting, exit cascade
2. `alpha-bot-15m.md` - 4 strategies, Supabase persistence, reference price alignment
3. `kalshi-trader-15m.md` - 111-commit OG build, external signals, bracket orders, hour bias
4. `kalshi-btcd-trader-hourly.md` - Terminal probability model (Black-Scholes), scored edge mode, circuit breakers

Also triaged and dismissed: kalshi-15m-scalper (stub), Dopealkshi (empty), kalshi-btcd-ai-trader (1-commit snapshot), compd-trader (UI only).

Updated CURRENT-STATE.md with full build inventory table, cross-build synthesis, strategy direction, and next steps.

### Rationale

The key finding: kalshi-btcd-trader's probability model (touchProb.ts) is the architectural blueprint for KXBTC15M. It computes fair value using Black-Scholes with per-minute sigma from Binance 1m candles, then trades when model probability diverges from market-implied probability. This is the only build that approaches trading as a probability-edge problem rather than a pattern-recognition problem.

Alpha-bot contributes the best infrastructure (Supabase, reference price alignment). kalshi-trader contributes external signals and hour-based bias. kalshi-ai-trader-v2 contributes the exit cascade structure.

All builds share two fatal flaws: no persistence and no exchange reconciliation.

### Consequences

- Research phase for KXBTC15M is COMPLETE. 7 adapted research files committed.
- Strategy direction is decided: probability-edge trading, Python, Supabase, paper mode first.
- Next step: Write `docs/STRATEGY-KXBTC15M.md` v2 as the full spec, then build code.
- CURRENT-STATE.md updated with full inventory and roadmap.
- Any new agent session will boot with complete context of all prior work.

---

## 2026-05-20 ~22:30 ET - Persistence & reconciliation guardrail system created

**Author:** Comet (browser agent) on behalf of meszaroszack
**Session:** Guardrail audit and gap closure
**Status:** decided
**Scope:** architecture | guardrails | docs

### Context

User asked: do we have documentation that ensures the two fatal flaws (no persistence, no exchange reconciliation) won't repeat? Audit revealed: STRATEGY-KXBTC15M.md sections 6-7 describe the right architecture, but no hard guardrail existed to block an agent from writing in-memory-only code. MASTER-PROMPT didn't list persistence/reconciliation as non-negotiable. CURRENT-STATE "Do Not Regress" omitted it.

### Decision

Created a 3-layer defense system:
1. `ai/guardrails/PERSISTENCE-RECONCILIATION.md` - Dedicated guardrail with 4 non-negotiable rules, 6-question self-test, error history table showing all 7 builds that failed
2. `ai/prompts/MASTER-PROMPT.md` v3 - Added as required read (#10) + 3 explicit non-negotiable rules in the non-negotiables block
3. `ai/handoffs/CURRENT-STATE.md` - Added 4 items to "Do Not Regress" list
4. `ai/checklists/PRE-CODE-REVIEW.md` - New pre-code review checklist with persistence/reconciliation/P&L/market/safety gates (all marked BLOCKING)

### Rationale

The prior builds didn't fail because developers didn't know about persistence. They failed because nothing enforced it. The strategy doc described the right architecture, but agents could ignore it. Now there are three independent enforcement points: the required read list, the non-negotiables block, and the pre-code checklist. An agent would have to violate all three to repeat the mistake.

### Consequences

- Any new agent session will read PERSISTENCE-RECONCILIATION.md before writing code
- MASTER-PROMPT explicitly forbids in-memory state and requires boot reconciliation
- Pre-code checklist gates code that touches trading/execution/state
- CURRENT-STATE "Do Not Regress" lists persistence and reconciliation as top-level requirements
- The two fatal flaws from all 7 prior builds are now documented, explained, and enforced at multiple levels


## 2026-05-21 19:00 ET - Kalshi API & WebSocket engineering research complete

**Author:** Perplexity Comet (research session)
**Session:** Kalshi API/WebSocket deep-dive and guardrail hardening
**Status:** decided
**Scope:** research | architecture | docs

### Context
User identified a massive oversight: no prior build had documented how to sustainably use Kalshi APIs and WebSocket, especially for order fulfillment and reconciliation. Existing guardrails (PERSISTENCE-RECONCILIATION.md Rules 1-4) covered boot reconciliation and Supabase persistence but had zero WebSocket or rate limit content.

### Decision
Conduct external research on Kalshi official docs, open-source trading bots, and community patterns. Produce research artifacts and harden guardrails with WebSocket reconciliation (Rule 5) and rate limit management (Rule 6).

### Rationale
Every prior build suffered from the same two fatal flaws (no persistence, no reconciliation). The guardrails addressed boot-time reconciliation but not real-time WebSocket reconciliation or rate limit engineering. This research closes those gaps.

### Consequences
- PERSISTENCE-RECONCILIATION.md now has 6 rules (was 4) and 10 self-test questions (was 6)
- Three new research/adapted/ deep-dives committed for future agent reference
- Any agent implementing trading code now has authoritative Kalshi API field mappings
- Rate limit awareness prevents 429-induced order failures

### Related files
- research/adapted/kalshi-public-docs-api-websocket.md (NEW)
- research/adapted/morningside-wagewise-orderbook.md (NEW)
- research/adapted/probablyprofit-order-management.md (NEW)
- ai/guardrails/PERSISTENCE-RECONCILIATION.md (UPDATED: Rules 5-6)
- ai/handoffs/CURRENT-STATE.md (UPDATED)
