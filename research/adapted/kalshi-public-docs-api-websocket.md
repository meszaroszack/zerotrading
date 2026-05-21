# Kalshi Public Docs - API & WebSocket Engineering

**Source URL:** https://docs.kalshi.com
**Commit / version:** Accessed 2026-05-21
**License:** Proprietary (Kalshi Exchange)
**Relevance:** Direct source-of-truth for all API and WebSocket integration patterns. Authoritative reference for order fulfillment, reconciliation, rate limits, and real-time data engineering.

---

## What We Extracted

### 1. WebSocket Architecture

**Connection:** Single authenticated WebSocket at `wss://api.elections.kalshi.com/trade-api/ws/v2`. All communication through one connection. Auth via API key headers during handshake is mandatory even for public channels.

**Critical channels for order fulfillment:**

| Channel | Type | Purpose |
|---------|------|---------|
| `user_orders` | Private | Real-time order state changes (created, resting, filled, canceled) |
| `fill` | Private | Individual fill events with trade-level detail |
| `orderbook_updates` | Public | Live orderbook deltas |
| `market_ticker` | Public | Price/volume updates |
| `market_positions` | Private | Position changes |
| `order_group_updates` | Private | Order group state changes |

**Keep-alive:** Kalshi sends Ping frames (0x9) every 10 seconds with body `heartbeat`. Clients MUST respond with Pong (0xA). Missing pongs causes disconnect.

### 2. User Orders Channel (Primary Reconciliation Feed)

Channel name: `user_orders`

Fires on every order state change. Message fields:
- `order_id` - Kalshi order identifier
- `user_id` - Your user ID
- `ticker` - Market ticker (e.g., FED-23DEC-T3.00)
- `status` - Current state: `resting`, `filled`, `canceled`
- `side` - `yes` or `no`
- `is_yes` - Boolean
- `yes_price_dollars` - Price in dollars (e.g., "0.3500")
- `fill_count_fp` - Contracts filled so far
- `remaining_count_fp` - Contracts still resting
- `initial_count_fp` - Original order size
- `taker_fill_cost_dollars` - Actual taker cost INCLUDING fees
- `maker_fill_cost_dollars` - Actual maker cost INCLUDING fees
- `client_order_id` - YOUR correlation key (idempotency)
- `order_group_id` - Group identifier for related orders
- `self_trade_prevention_type` - e.g., `taker_at_cross`
- `created_time` - ISO timestamp
- `expiration_time` - Order expiry ISO timestamp
- `subaccount_number` - Subaccount

**Key insight:** `taker_fill_cost_dollars` and `maker_fill_cost_dollars` include Kalshi fees. Use these for P&L, not price * quantity.

### 3. User Fills Channel (Trade-Level Granularity)

Channel name: `fill`

Fires immediately when orders are filled. Each message is one individual fill:
- `trade_id` - Unique trade identifier
- `order_id` - Which order this fill belongs to
- `market_ticker` - Market ticker
- `is_taker` - Boolean
- `side` - `yes` or `no`
- `yes_price_dollars` - Fill price
- `count_fp` - Contracts in this fill
- `action` - `buy` or `sell`
- `ts` - Unix timestamp (seconds)
- `ts_ms` - Unix timestamp (milliseconds)
- `post_position_fp` - Your position after this fill
- `purchased_side` - Which side you now hold
- `subaccount` - Subaccount number

**Key insight:** Subscribe to BOTH `user_orders` AND `fill`. They serve different purposes: `user_orders` gives order-level state transitions; `fill` gives trade-level granularity for P&L.

### 4. Rate Limits & Token Budgets

Two independent budgets:
- **Read bucket:** GET endpoints
- **Write bucket:** Order placement, amends, cancels, order groups, RFQ

**Tier budgets (tokens per second):**

| Tier | Read | Write |
|------|------|-------|
| Basic | 200 | 100 |
| Advanced | 300 | 300 |
| Premier | 1,000 | 1,000 |
| Paragon | 2,000 | 2,000 |
| Prime | 4,000 | 4,000 |

Most requests cost 10 tokens. Cancellations and single-order reads are cheaper.

**Batch endpoints do NOT save tokens.** 25 orders in batch = 250 tokens.

**Burst capacity:** Write bucket holds 2 seconds of budget (Basic = 1 second).

**On 429:** Exponential backoff until bucket refills.

**Engineering implication:** Use WebSocket for real-time data. REST for boot reconciliation and periodic snapshots only.

### 5. Market Lifecycle & Settlement

States: `initialized` -> `active` -> `closed` -> `determined` -> `finalized`

- `initialized`: Created, not yet open. Transitions to `active` at `open_time`.
- `active`: Open for trading.
- `inactive`: Temporarily paused by exchange.
- `closed`: Past `close_time`. No new orders. Resting orders auto-cancelled (cancellation updates on WebSocket).
- `determined`: Outcome known. Settlement timer running (`settlement_timer_seconds`). Result may be disputed.
- `finalized`: Settlement complete. Positions paid out. Terminal state.

Settlement: Yes=$1 on yes outcome; No=$1 on no outcome. Net positions only. Zero fees for simple yes/no settlement. WebSocket `settled` event maps to REST `finalized` status.

### 6. Historical Data Boundary

Live endpoints cover ~3 months. Older records need `GET /historical/...` endpoints.
- Check `GET /historical/cutoff` for boundary timestamps
- For complete fill history: merge live + historical results

### 7. Orderbook Structure

Kalshi orderbooks return bids only (not asks). Binary market reciprocity:
- YES BID at X = NO ASK at ($1.00 - X)
- NO BID at Y = YES ASK at ($1.00 - Y)

---

## How This Applies to ZeroTrading

### Boot Reconciliation (maps to PERSISTENCE-RECONCILIATION.md Rule 2)
1. REST `GET /portfolio/orders` for all open orders
2. REST `GET /portfolio/positions` for all positions
3. REST `GET /portfolio/fills` for recent fills
4. Compare against Supabase state
5. Resolve discrepancies BEFORE entering trading loop

### Real-Time Reconciliation (NEW - gap in prior guardrails)
1. Subscribe to `user_orders` + `fill` WebSocket channels
2. Every fill event -> atomic Supabase write BEFORE acting on it
3. Use `client_order_id` as correlation key
4. Track `fill_count_fp` vs `initial_count_fp` for partial fills
5. Use `taker_fill_cost_dollars` / `maker_fill_cost_dollars` for real P&L

### Periodic Safety-Net Reconciliation (NEW)
1. Every 60s: full REST snapshot of orders + positions
2. Compare against WebSocket-derived Supabase state
3. Alert on drift; auto-correct if discrepancy found

### Rate Limit Budget Management (NEW)
1. Track token consumption per bucket (read/write)
2. Pre-check available tokens before API calls
3. Exponential backoff with jitter on 429s
4. WebSocket reconnection: exponential backoff 1s -> 60s max

---

## DO / DO NOT

**DO:**
- Use `client_order_id` on EVERY order (idempotency + correlation)
- Subscribe to BOTH `user_orders` AND `fill` channels
- Persist every fill to Supabase BEFORE acting on it
- Handle `determined` -> `finalized` settlement window
- Use WebSocket for real-time; REST for reconciliation snapshots
- Respond to Ping frames within 10 seconds
- Re-bootstrap state on WebSocket reconnect

**DO NOT:**
- Rely solely on WebSocket (always have REST reconciliation backup)
- Assume batch endpoints save rate limit tokens (they do not)
- Ignore partial fills (`remaining_count_fp` can be non-zero)
- Calculate P&L from price alone (must include maker/taker costs)
- Place orders before boot reconciliation completes
- Treat `closed` as `settled` (determination + dispute window exists)
- Hardcode market tickers (KXBTC15M and KXBTCD have different lifecycles)
