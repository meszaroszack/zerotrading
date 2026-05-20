# GUARDRAIL: Kalshi BTC Market Reference

> **REQUIRED READING for every AI agent working on this repo.**
> **This document defines the different Kalshi BTC contract types and their structures.**
> **Each system in this repo targets a SPECIFIC market. Do not confuse them.**

---

## Why This Document Exists

Kalshi offers MULTIPLE Bitcoin contract types with DIFFERENT structures. AI agents and developers consistently confuse them, applying the wrong mental model to the wrong market. This leads to fundamentally broken strategies.

Before writing ANY code or strategy, you MUST know which market you're targeting and how that specific market works.

---

## The Two Markets ZeroTrading Targets

### 1. KXBTCD - Hourly BTC Price Threshold ("Above/Below")

**Ticker format:** `KXBTCD-{DATE}-T{PRICE}` (e.g., `KXBTCD-26MAY2001-T76699.99`)
**Settlement:** Every hour (e.g., 1:00 AM, 2:00 AM, ...)
**Currently deployed at:** `zerotrading-core-production.up.railway.app`

#### How it works

1. Kalshi publishes a **ladder of strike prices** for each hourly window (e.g., $76,500, $76,600, $76,700, $76,800...).
2. For EACH strike on the ladder, there is a separate contract asking: "Will BTC be above or below THIS price at settlement?"
3. You pick a strike from the ladder and buy YES (above) or NO (below) on that specific threshold.
4. Multiple contracts exist simultaneously for the same hour at different price levels.
5. Settlement = average of final 60 seconds of CFB Real-Time Index before the hour ends.

#### Key characteristics

- **Multiple strikes per window** - you choose WHERE on the ladder to trade.
- **Strike distance matters** - how far the strike is from current price determines probability and risk/reward.
- "Strike too close" or "strike too far" are valid strategy concepts here.
- The bot evaluates implied probability vs estimated probability at a specific price level.
- Contracts settle hourly.
- This IS the market where "price windows," "strike distance," and "threshold selection" logic apply.

#### What the live bot (Railway) does

- Monitors the KXBTCD orderbook for the nearest hourly settlement.
- Evaluates the contract at the strike closest to current BTC price.
- Computes strike distance, implied probability, net edge.
- Enters if edge exceeds threshold; manages through an execution FSM (IDLE -> ENTERING -> OPEN -> EXITING -> SETTLED).

---

### 2. KXBTC15M - 15-Minute BTC Up/Down ("Over/Under")

**Ticker format:** TBD (varies by window)
**Settlement:** Every 15 minutes
**Status:** Research/paper mode. No live deployment yet.

#### How it works

1. Every 15 minutes, a new contract opens.
2. The strike = BTC spot price at the moment the window opens.
3. ONE question: "Will BTC be ABOVE or BELOW this strike at the end of 15 minutes?"
4. You buy YES (up/over) or NO (down/under).
5. ONE strike per window. No ladder. No choosing.
6. Settlement = average of final 60 seconds of CFB RTI before the 15m window ends.

#### Key characteristics

- **Single strike per window** - no ladder, no choice of price level.
- Strike distance is NOT a concept here - the strike IS the current price.
- "Price windows" and "brackets" do NOT apply.
- This is a pure directional bet: will BTC go up or down in the next 15 minutes?
- Contracts settle every 15 minutes.
- This is NOT the market where range/wedge/convergence logic applies.

**See `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md` for detailed guardrails specific to this market.**

---

## Side-by-Side Comparison

| Feature | KXBTCD (Hourly) | KXBTC15M (15-min) |
|---|---|---|
| Settlement frequency | Hourly | Every 15 minutes |
| Strikes per window | Multiple (ladder) | ONE (current price) |
| Strike selection | Choose from ladder | Automatic (spot price) |
| Contract question | "Above/below $X?" | "Up or down?" |
| Strategy concepts | Strike distance, threshold selection, price windows | Directional momentum, up/down prediction |
| Strike distance relevant? | YES - core concept | NO - not applicable |
| Range/bracket applicable? | Partially - you pick level | NO |
| Live deployment | YES - Railway app | NO - paper mode |
| Ticker prefix | KXBTCD | KXBTC15M |
| Settlement method | Avg of final 60s CFB RTI | Avg of final 60s CFB RTI |

---

## Other Kalshi BTC Markets (Not Currently Targeted)

Kalshi also offers these BTC markets which ZeroTrading does NOT currently target:

### BTC Price Range (Hourly)
- Question: "What range will BTC price fall in at hour end?" (e.g., $76,600 to $76,699.99)
- This IS a true range/bracket product with mutually exclusive price bands.
- NOT the same as KXBTCD above/below threshold contracts.

### BTC Daily Price
- Longer-term contracts on daily closing prices.
- Settlement at end of day.

### BTC Weekly/Monthly
- Even longer-term prediction contracts.

---

## Common AI Errors

| Error | Which market it applies to | Why it's wrong |
|---|---|---|
| "KXBTC15M has price windows/brackets" | Neither | 15M is simple up/down; no windows |
| "KXBTCD is a simple over/under" | Partially true | KXBTCD IS over/under, but at CHOSEN price levels from a ladder |
| "Strike distance doesn't matter for KXBTCD" | Wrong | Strike distance is core to KXBTCD strategy |
| "KXBTC15M has strike distance" | Wrong | 15M strike = current price; distance is always zero at open |
| Applying convergence/range logic to KXBTC15M | Wrong | 15M is directional, not range-based |
| Applying 15M up/down logic to KXBTCD | Wrong | KXBTCD requires threshold selection from the ladder |
| Confusing KXBTCD with BTC Price Range | Wrong | They're different contract types even though both are hourly |

---

## Settlement Details (Both Markets)

From Kalshi docs: All crypto contracts settle using the **average of 60 seconds of CFB Real-Time Index** values before the settlement time. This means:

- Settlement begins 1 minute BEFORE the stated expiration time.
- The settlement price is an average, not a point-in-time snapshot.
- A spike in the final seconds can affect settlement but is smoothed by the 60-second average.
- Example: A contract settling at 1:00 AM uses the average of BTC prices from 12:59:00 AM to 12:59:59 AM.

---

## Fee Structure (Both Markets)

From the Kalshi fee schedule:

```
fees = round_up(0.07 * C * P * (1-P))

P = contract price in dollars (50 cents = 0.5)
C = number of contracts
```

- Fees are highest when P is near 0.50 (50/50 contracts).
- Fees decrease as P approaches 0 or 1 (high-confidence bets).
- **Maker orders (limit orders) incur NO fees if filled by a taker.**
- **No settlement fees** - you only pay trade execution fees.
- This fee structure applies to BOTH KXBTCD and KXBTC15M.

---

## How This Affects the Repo

| System | Market | Key files |
|---|---|---|
| ZeroTrading Core (Railway) | KXBTCD (hourly) | Deployed live. Architecture in `docs/pdfs/`. |
| ZeroTrading 15M (planned) | KXBTC15M (15-min) | `docs/STRATEGY-KXBTC15M.md`, `research/mystic/` |

**Do NOT mix strategies between markets.** KXBTCD strategies (strike distance, threshold selection) do not apply to KXBTC15M, and vice versa.

---

## Related Files

- `ai/guardrails/KXBTC15M-MARKET-STRUCTURE.md` - 15M-specific guardrails.
- `docs/STRATEGY-KXBTC15M.md` - 15M strategy spec (v3, over/under).
- `docs/pdfs/` - Core architecture docs (applies to KXBTCD live system).
- `ai/handoffs/CURRENT-STATE.md` - Live pointer.
