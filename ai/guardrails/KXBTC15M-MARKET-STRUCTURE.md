# GUARDRAIL: KXBTC15M Market Structure

> **REQUIRED READING for every AI agent working on this repo.**
> **This file must be referenced in every prompt, handoff, and fresh-session template.**
> **If you are an AI agent and you have not read this file, STOP and read it now.**

---

## The #1 Mistake AI Makes on KXBTC15M

AI models consistently hallucinate additional complexity onto KXBTC15M contracts. This has happened multiple times in this project and is a **known, recurring failure mode** that must be actively guarded against.

### What KXBTC15M actually is

KXBTC15M is a **simple binary over/under contract.**

1. Every 15 minutes, a new contract opens.
2. The **strike price** = BTC spot price at the moment the contract opens.
3. The ONLY question: **"Will BTC be ABOVE or BELOW this strike price when the 15 minutes are up?"**
4. You buy **YES** (above/over) or **NO** (below/under).
5. At settlement: if BTC is above strike, YES = $1.00, NO = $0.00. If below, reverse.

That's it. One price. Two outcomes. Over or under.

### What KXBTC15M is NOT

AI agents have incorrectly described KXBTC15M as ALL of the following. **None of these are correct:**

- A range or band product ("will BTC stay within +/-$250")
- A wedge or convergence product
- A volatility product
- A multi-strike ladder with multiple brackets
- A product with strike distances or strike ranges
- A product where you bet on BTC "staying in range"
- A product with convergence zones
- Anything involving the phrase "stay in range"

### Why AI gets this wrong

1. **Training data contamination:** AI models have seen options, futures, and other complex derivatives in training. They pattern-match KXBTC15M to these more complex products.
2. **Over-engineering bias:** AI tends to add complexity. A simple over/under doesn't "feel" sophisticated enough, so the model invents range mechanics.
3. **Confusion with other Kalshi products:** Kalshi has other contract types (hourly ranges, daily high/low) that ARE range products. AI confuses these with KXBTC15M.
4. **Hallucination under uncertainty:** When the AI isn't sure about the exact contract structure, it fills in gaps with plausible-sounding but wrong mechanics.

### How to prevent this error

1. **Every strategy document** must include a section confirming the over/under structure.
2. **Every code review** must check that the logic is binary (above/below strike), not range-based.
3. **Every AI-generated analysis** must be reviewed for wedge/range/band language. If found, the analysis is wrong.
4. **This file** must be included in or referenced by:
   - `ai/prompts/MASTER-PROMPT.md`
   - `.cursor/rules/` (for Cursor IDE agents)
   - Every fresh-session handoff template
   - Every strategy document header

---

## Quick Self-Test for AI Agents

Before writing any KXBTC15M strategy or analysis, answer these questions:

1. How many outcomes does a KXBTC15M contract have? **Answer: 2 (above or below strike)**
2. What determines the strike price? **Answer: BTC spot price at the start of the 15m window**
3. Is there a range or band involved? **Answer: NO**
4. Can you buy a "stays in range" position? **Answer: NO**
5. What are the two sides you can buy? **Answer: YES (over/above) or NO (under/below)**

If you answered any of these differently, re-read this document.

---

## Error History

| Date | Error | Where | Fix |
|---|---|---|---|
| 2026-05-20 | Described KXBTC15M as range/wedge/convergence product | STRATEGY-KXBTC15M.md v2, mystic-performance-summary.md v1, PERFORMANCE-RECORDS.md v1 | Corrected to v3 strategy, v2 Mystic summary. All docs updated. |

This table should be updated every time this error recurs.

---

## Files That Reference This Guardrail

- `docs/STRATEGY-KXBTC15M.md` - Section "CRITICAL: KXBTC15M Market Structure"
- `research/mystic/mystic-performance-summary.md` - Section "IMPORTANT: KXBTC15M is Over/Under"
- `ai/prompts/MASTER-PROMPT.md` - Should reference this file
- `.cursor/rules/` - Should reference this file
