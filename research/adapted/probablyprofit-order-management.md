# ProbablyProfit - Order Management & Reconciliation Patterns

**Source URL:** https://github.com/randomness11/probablyprofit
**Commit / version:** Accessed 2026-05-21
**License:** Unknown (public repo)
**Relevance:** AI-powered trading bot for Polymarket with a mature OrderManager pattern, partial fill tracking, event-driven callbacks, and built-in reconciliation. The order management architecture is directly applicable to our Kalshi integration despite targeting a different exchange.

---

## What We Extracted

### Architecture Overview

Three-layer pipeline:
1. **AI Decision Engine** - Multi-model ensemble (Claude, GPT-4, Gemini) with 2/3 vote consensus
2. **Risk Management** - Kelly criterion sizing, position limits, stop-loss, drawdown protection
3. **Order Management** - Full lifecycle management interfacing with Polymarket CLOB API

### OrderManager Pattern

The `OrderManager` is the central class governing order lifecycle:

**Core capabilities:**
- Partial fill tracking (not just all-or-nothing)
- Event callbacks: `on_fill`, `on_complete`, `on_cancel`
- State reconciliation against exchange on every cycle
- Telegram alerts for reconciliation drift

**Event-driven design:**
- Orders are submitted and tracked by internal ID
- Fill events fire callbacks that update local state
- Reconciliation compares local vs exchange state and alerts on mismatch

### Reconciliation Integration

Reconciliation is NOT bolted on as an afterthought. It is integrated directly into the OrderManager:

1. Every order cycle includes a reconciliation check
2. Drift between local and exchange state triggers alerts (Telegram)
3. Categories of reconciliation issues are enumerated and handled distinctly
4. The system can detect: missed fills, phantom orders, balance drift, position mismatch

### Risk Management Layer

- **Kelly criterion** for position sizing
- **Position limits** per market and aggregate
- **Stop-loss** at individual trade level
- **Drawdown protection** at portfolio level
- Risk parameters are configurable, not hardcoded

---

## How This Applies to ZeroTrading

### Patterns to Adopt

1. **OrderManager as first-class module** - Not embedded in strategy logic. Separate module with clean API: `submit_order()`, `cancel_order()`, `get_order_status()`, `reconcile()`.

2. **Event callbacks for fill handling** - `on_fill` callback triggers Supabase write + P&L update. `on_complete` triggers position state update. `on_cancel` triggers cleanup.

3. **Reconciliation as part of the order cycle, not separate** - Every N seconds (or every strategy cycle), the OrderManager self-reconciles against exchange state. Not a separate cron job or manual process.

4. **Alerting on reconciliation drift** - When local state diverges from exchange state, immediately alert (Telegram, webhook, or Supabase log). Do not silently correct.

5. **Kelly criterion sizing** - Mathematical position sizing based on edge and bankroll, not fixed contract counts. Maps to our existing sizing logic in STRATEGY-KXBTC15M.md.

### Patterns NOT to Adopt

1. **Polymarket-specific CLOB interaction** - Different exchange, different API. Use Kalshi-specific patterns from kalshi-public-docs-api-websocket.md.
2. **Multi-model AI ensemble for decisions** - Our strategy is probability-model-based per STRATEGY-KXBTC15M.md, not LLM voting.

---

## DO / DO NOT

**DO:**
- Build OrderManager as a standalone module with clean interfaces
- Implement event callbacks (on_fill, on_complete, on_cancel)
- Integrate reconciliation INTO the order management cycle
- Alert on reconciliation drift before auto-correcting
- Use Kelly or fractional-Kelly for position sizing

**DO NOT:**
- Embed order management logic inside strategy code
- Silently auto-correct state drift without logging/alerting
- Use fixed contract sizes instead of mathematical sizing
- Skip partial fill tracking (assume all-or-nothing)
