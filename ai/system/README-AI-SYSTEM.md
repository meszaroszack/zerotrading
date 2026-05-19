# AI System Overview

This directory contains canonical instructions, rules, and specifications for all AI agents working on ZeroTrading. Every AI agent — Perplexity Computer, Cursor, Claude Code, or any other — must treat these files as authoritative project context before making any changes.

## Read Order

1. This file
2. `MODEL-SPEC.md` — behavior, constraints, and output expectations
3. `OPEN-OPSEC-POLICY.md` — what is public, what is private
4. `REPO-PRINCIPLES.md` — documentation-first and open-knowledge principles

## Purpose

- Align all agents on product vision, architecture, and safety constraints.
- Reduce session drift by providing stable written guidance that persists across conversations.
- Make AI contributions reproducible, reviewable, and auditable via GitHub.

## Mandatory Behaviors for All AI Agents

- Read the MODEL-SPEC before writing any code.
- Check `ai/handoffs/FRESH-SESSION-TEMPLATE.md` for current project state.
- End every session with a DECISION-LOG entry and a fresh-session summary.
- Never commit secrets or bypass the protected execution/accounting core.
- Push all documentation changes directly to GitHub, not just to local or chat context.

## Directory Map

```
ai/
  system/          <- You are here. Canonical agent rules.
  prompts/         <- Copy-paste prompts for implementation passes.
  handoffs/        <- Fresh-session templates and current state.
  checklists/      <- Session output contracts and safety checks.
  evals/           <- Eval cases for strategy, accounting, and agent output.
  schemas/         <- JSON schemas for structured agent outputs.
  summaries/       <- Decision log and per-session summaries (timestamped).
```
