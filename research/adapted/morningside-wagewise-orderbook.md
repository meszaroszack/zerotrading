# Morningside WageWise - L3 Orderbook Architecture

**Source URL:** https://github.com/amirzarandi/morningside-wagewise
**Commit / version:** Accessed 2026-05-21
**License:** Unknown (public repo)
**Relevance:** C++ L3 orderbook engine built specifically for Kalshi prediction markets. Demonstrates high-performance local orderbook management patterns applicable to our order tracking and reconciliation needs.

---

## What We Extracted

### Architecture Overview

The project maintains a local orderbook replica for Kalshi markets, abstracting prediction market order flow into traditional bid/ask structures.

**Data structures:**
- `std::map` for price levels (sorted)
- `std::list` for FIFO ordering within price levels
- `std::unordered_map` for O(1) order lookups by order ID

**Performance benchmarks (Google Benchmark):**

| Operation | Throughput |
|-----------|-----------|
| Order adds | ~823,000/sec |
| Order cancels | ~2,700,000/sec |
| Mixed workload | ~813,000/sec |
| Full matches (100 orders) | ~26,000/sec |

**Complexity:**
- Add orders: O(log n) (tree insertion)
- Cancel orders: O(1) (hash map lookup)
- Order info queries: O(1) (hash map lookup)

### Current State

- REST-only data feed (HTTP GET for orderbook snapshots from Kalshi)
- WebSocket integration identified as primary next step
- Architected to support multiple exchanges (Polymarket, etc.)
- Custom C++ market data feed handler, separate from matching engine

### Key Design Decisions

1. **Separation of data feed handler from matching engine** - The feed handler fetches/receives market data; the engine processes it. Clean boundary.
2. **O(1) order lookups** - `unordered_map` keyed by order ID means instant lookup for cancel/modify operations. Critical for reconciliation.
3. **Local replica, not passthrough** - Maintains full local state, enabling fast decisions without round-trips to exchange.

---

## How This Applies to ZeroTrading

### Patterns to Adopt

1. **O(1) order tracking by ID** - Our Supabase order table should be indexed by both `order_id` (Kalshi's) and `client_order_id` (ours) for instant lookups during WebSocket fill events.

2. **Separate data feed from decision engine** - Keep the WebSocket connection manager and message parser completely separate from strategy logic. Feed handler writes to Supabase; strategy reads from Supabase.

3. **Local orderbook for spread/depth awareness** - Even without building a full matching engine, maintaining a local orderbook snapshot (from WebSocket `orderbook_updates`) enables real-time spread and depth checks before placing orders.

### Patterns NOT to Adopt

1. **C++ implementation** - We are Python-first per MASTER-PROMPT. The architectural patterns are what matter, not the language.
2. **REST-only data feed** - They acknowledge this is a limitation. We should go straight to WebSocket per kalshi-public-docs-api-websocket.md.
3. **No persistence layer** - This repo has no database. All state is in-memory. We MUST persist to Supabase per PERSISTENCE-RECONCILIATION.md.

---

## DO / DO NOT

**DO:**
- Index orders by both exchange ID and client ID for O(1) lookups
- Separate data ingestion from strategy logic
- Maintain local orderbook state for spread/depth awareness
- Benchmark critical-path operations

**DO NOT:**
- Build a full matching engine (Kalshi handles matching; we just need local state tracking)
- Use in-memory-only state (must persist to Supabase)
- Skip WebSocket in favor of REST polling for real-time data
