# Market Data Resilience

> **Required reading before touching any code that reads current price, bid, ask, orderbook, or any other live market signal.**

This guardrail exists because of FIX-CORE-010 (2026-05-23): a transient `null` from `adapter.getOrderbook()` silently disabled TP/SL exit logic for entire strategy ticks in `zerotrading-core`. The bot relied on hour-close settlement to rescue positions any time Kalshi's REST endpoint blipped. Operator observation: "P&L math has never really worked." The bug had been present in every shipped build of the model.

The lesson generalizes well beyond this one bug. It is now a hard architectural rule.

---

## The Rule

Any code path that reads "current price," "current bid/ask," or "current orderbook" from a single live source is wrong by default. **Read from at least two independent sources** and synthesize a fallback when the primary returns null, empty, or stale.

The bar:

> If Kalshi REST returns 500 for 30 seconds, does TP/SL still fire? If the answer is "we'd wait for the next tick" or "we'd settle at hour-close," **that is a bug.**

Trading logic must degrade gracefully, not silently. A momentary network failure must not change a position's exit behavior.

---

## Approved patterns

### Pattern A — Live + summary fallback (REST-only architectures)

Used by `zerotrading-core` after FIX-CORE-010. Two REST endpoints, both polled per strategy tick:

1. **Primary:** `GET /markets/{ticker}/orderbook` — full depth, fresh.
2. **Fallback:** `GET /markets?series_ticker=...` — `KalshiMarketSummary` includes top-of-book `no_bid_cents`, `no_ask_cents`, `yes_bid_cents`, `yes_ask_cents`.

When the primary returns `null` or `noBid <= 0`, synthesize a minimal `KalshiOrderbookView` from the summary (`synthesizeOrderbookFromSummary()` in `src/strategy/endOfHour.ts`). Depth is zero, but top-of-book pricing is good enough to drive TP/SL thresholds and urgent-exit price computation.

Only conclude "market is actually gone" when **both** sources indicate so (summary status is `closed`/`settled` OR `no_bid_cents == 0`).

### Pattern B — WS + REST refresher (event-driven architectures)

Used by `kalshi-15m-bot` (Python). Two concurrent loops in `asyncio.gather`:

1. **Primary:** WebSocket subscription emits orderbook updates and updates an in-memory `state` object.
2. **Fallback:** REST poller fires every 10 s; for any ticker whose `state.last_updated` is older than 15 s, it does a one-shot REST fetch and overwrites `state.no_bid`, `state.yes_bid`, etc.

Consumers read from `state.*` only — never directly from the WebSocket buffer or directly from REST. `state` is **never null** by construction; the worst case is briefly stale.

### Pattern C — Entry-time stamp (positions only)

Used by `kalshi-btcd-trader/server/tradingEngine.ts:2383`. For data that does not change after entry (avg fill price, fee paid, position open timestamp), stamp it onto the `Position` record at fill time. Never compute it from live data later.

This is **not** a substitute for A or B — it covers a different category of data. Combine: stamp entry-time invariants at fill; read tick-by-tick prices from a resilient source.

---

## Prohibited patterns

### ❌ Single-source gate

```ts
const orderbook = await adapter.getOrderbook(ticker);
if (orderbook === null) return;  // ← BUG. Silently disables everything downstream.
// ... TP/SL, UI fields, audit logs all live below this line.
```

This is the original `zerotrading-core` bug. **Never write this.**

If you find yourself writing `if (orderbook === null) return;` in any function that runs per strategy tick, stop and ask: what work below this line just got silently skipped on a transient blip?

### ❌ Single source of truth for UI display

```ts
return {
  avgEntryPriceCents: snap.exitPosture?.avgEntryPriceCents ?? null,
  currentNoBidCents: snap.exitPosture?.currentNoBidCents ?? null,
  // ... seven more fields all reading from the same nullable object.
};
```

When the upstream computation fails, every downstream field collapses to `—` on the dashboard. The operator loses observability into the position at the exact moment trading logic is also impaired.

Use the multi-source helper pattern (`derivePositionUiFields()` in `src/server/index.ts`): try the rich object first, fall back to persisted records and the live market summary independently for each field.

### ❌ Coupling display data to decision data

The same `exitPosture` object that drives TP/SL must not also be the only source of dashboard fields. They have different failure tolerance:

- Decision data: must be fresh and correct, or we make a wrong trade.
- Display data: must be present and approximately right, or the operator loses visibility.

A momentary loss of decision-quality data does not justify blanking the dashboard. The operator may still want to see the last-known avg entry and bid even if a TP/SL evaluation was skipped this tick.

---

## Pattern-propagation rule (cross-repo)

When a market-data resilience pattern is established in one repo, **audit every other repo with the same broker integration** for the same anti-pattern. FIX-CORE-010 existed in `zerotrading-core` for the lifetime of the project because the team already had Pattern B in `kalshi-15m-bot` and Pattern C in `kalshi-btcd-trader` — but never propagated the lesson back to the production hourly bot.

**Concretely:** before declaring a market-data PR complete, run this audit in every other repo that talks to the same broker:

```bash
# Cross-repo: any single-source orderbook gate that early-returns?
rg "getOrderbook|get_orderbook" --type ts --type py -A 3 | rg -B 1 "if.*null|if.*None"
```

If matches return, those are candidate bugs. Open issues for each before closing the originating session.

---

## Detection — audit signals to wire up

When fallback is in effect, emit a structured signal so chronic broker instability is observable:

- `zerotrading-core` emits `orderbookFromFallback: boolean` on every `URGENT_EXIT_TICK` audit log entry (FIX-CORE-010 PR #5). Expected steady-state rate: 0%.
- A non-zero rate over any 1-hour window indicates REST instability. The bot is still functional (fallback is correct by construction), but operator should investigate broker connectivity.

**Open follow-up across repos:** wire these signals to a counter and surface on `/api/status` (or equivalent) so the operator does not have to grep audit logs.

---

## Tests required for any market-data code path

When adding or modifying code that reads market data, the PR must include tests covering:

1. **Happy path** — both sources return fresh data; primary wins.
2. **Primary down, fallback usable** — primary returns null/empty; fallback synthesized; downstream logic still runs.
3. **Both down, market actually closed** — both sources unusable; reconcile-as-finalized branch fires.
4. **Both down, market actually open (paranoid)** — both sources unusable but market summary `status === 'open'`; what does the code do? Document the answer.

These four cases pin every branch of the data-source decision tree. Without them, the next regression is invisible until production.

---

## Cross-references

- Origin incident: `ai/summaries/CRASH-AND-FIX-LOG.md` → **FIX-CORE-010**
- Display-side fix: `zerotrading-core` PR #4 (`f809f18`) — `derivePositionUiFields()`
- Decision-side fix: `zerotrading-core` PR #5 (`6e495cf`) — `synthesizeOrderbookFromSummary()`
- Sibling reference patterns:
  - `meszaroszack/kalshi-btcd-trader` — Pattern C (entry-time stamps)
  - `meszaroszack/kalshi-15m-bot` — Pattern B (WS + REST refresher)
- Forward-looking risk: `zerotradingx15minbtc` has no exit/TP/SL logic implemented yet. When it is added, **this guardrail must be reviewed first**. The entry-side null check at `src/strategy/kxbtc15m.py:232` is benign (causes NO_TRADE) but the exit-side equivalent would replicate FIX-CORE-010 verbatim.
