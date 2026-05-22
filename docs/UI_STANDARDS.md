# UI Standards — ZeroTrading Platform

**Status:** REQUIRED — applies to every operator-facing interface across all bots and repos.  
**Scope:** Any HTML dashboard, SPA, or monitoring UI served by any bot in this platform.

---

## Non-Negotiables

Every bot in this platform **must ship an HTTP dashboard**. A trading bot with no UI is a black box. Operators cannot monitor, pause, or diagnose without visibility. A dashboard is not optional — it is a baseline requirement, equivalent to structured logging.

---

## Design System

### Colors

| Token | Hex | Use |
|-------|-----|-----|
| Background | `#07090f` | Page background |
| Surface | `#0e1118` | Top bar, sidebar, status bar |
| Card | `#131722` | Content cards |
| Border | `#1e2538` | All borders and dividers |
| Accent (Blue) | `#3b82f6` | Active state, links, primary CTAs |
| Success (Green) | `#22c55e` | WIN, LIVE, profitable P&L |
| Danger (Red) | `#ef4444` | LOSS, DOWN, error, negative P&L |
| Warning (Yellow) | `#f59e0b` | Paused state, caution |
| Text | `#e2e8f0` | Primary body text |
| Muted | `#64748b` | Labels, secondary text, placeholders |

### Typography

- Font stack: `'Inter', system-ui, -apple-system, sans-serif`
- All numeric values must use `font-variant-numeric: tabular-nums` — numbers must not shift width as they update
- Do not use variable-width fonts on prices, P&L, contracts, timestamps, or any live-updating number

### Layout

- Dark sidebar navigation on the left, content on the right
- Top bar: logo | uptime | FSM state | BTC spot | mode | spacer | pause button
- Bottom status bar: last updated | feed health pills | active ticker | mode
- Minimum 5 pages: Overview, Live Position, Decision Log, Trade History, Bot Health

---

## Content Standards

### Zero Dead Elements

Every element visible on screen must be functional or have a plain-English explainer. This means:

- **No placeholder text.** If data isn't available yet, say why: "Waiting for orderbook data…" not "—" alone.
- **No coming-soon elements.** Do not render a button, card, or section that does nothing.
- **No empty states without explanation.** If no position is open, explain it and show the last decision.

### Every Number Needs a Label

Every numeric value displayed must have:
1. A human-readable label explaining what it measures (not just the field name)
2. A `title` attribute or inline description explaining the unit and significance

Examples:
- ✅ `Fill Price (¢)` with `title="Price paid per contract in cents. Max payout is 100¢."` 
- ✅ `Edge` with `title="Edge = model probability minus Kalshi market price. Higher edge = more conviction."`
- ❌ `fill_price` with no context

Traders and non-developers must understand every value shown without reading source code.

### Feed Health Pills

Feed health indicators must always show one of two states: `LIVE` (green) or `DISCONNECTED` (red). Grey/unknown is not a valid display state — if status is unknown, show DISCONNECTED.

---

## Technical Requirements

### Server

- HTTP dashboard served on `PORT` env var (Railway injects this), default `8080`
- Pure stdlib only (`http.server`, `threading`, `json`) — no Flask, FastAPI, Streamlit, Gunicorn
- No new pip dependencies for the dashboard

### Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Full SPA HTML |
| `/api/status` | GET | JSON snapshot, polled every 3s by the SPA |
| `/health` | GET | `200 ok` — Railway health check, no DB call |
| `/api/pause` | POST | Toggle pause flag, returns `{"paused": bool}` |
| `/api/export` | GET | CSV export of positions |

### Performance

- `/health` must return `200` within 100ms — no DB call, no FSM call
- `/api/status` must never block on a live DB query — all Supabase data must come from a background-refreshed cache (5s TTL)
- SPA must poll `/api/status` every 3000ms and update elements in-place (no page reload)
- Show a red "Connection lost" banner after 3 consecutive fetch failures

### Pause Behavior

- `POST /api/pause` toggles a module-level `_paused` flag
- The trading loop checks `is_paused()` at the top of every iteration and skips evaluation if True
- Dashboard shows a yellow banner while paused: "BOT IS PAUSED — No new trades will be evaluated until resumed"

### Security

- Env var values are **never** sent to the frontend. The `/api/status` endpoint returns only `true/false` for whether each variable is set.
- No `localStorage` or `sessionStorage` (Railway sandbox blocks these)

### Railway Integration

```toml
[deploy]
healthcheckPath = "/health"
healthcheckTimeout = 10
```

---

## Required Dashboard Pages

### Overview
- 4 KPI cards: Today P&L, Win Rate, FSM State, BTC Spot
- Each KPI has a one-line plain-English description below the value
- Cumulative P&L line chart (Chart.js CDN) from settled positions
- Active market card: ticker + window countdown + YES/NO probability bar
- Feed health pills: Kalshi WS + Binance WS

### Live Position
- IDLE state: large "NO OPEN POSITION" + last decision reason + time of last evaluation
- POSITIONED/SETTLING: full position detail — ticker, side, contracts, fill price, unrealized P&L, countdown, FSM state description
- FSM state descriptions in plain English (see below)

### Decision Log
- All decisions from Supabase `decisions` table, last 50, newest first
- Columns: Time, Window Ticker, Is Trade, Side, Edge, Vol Regime, Reason
- Filter buttons: ALL / TRADES ONLY / SKIPPED ONLY
- Table header explainer above

### Trade History
- Settled positions from Supabase `positions` table, last 50
- Columns: Opened, Ticker, Side, Contracts, Fill Price, Result, P&L
- P&L green/red. Running total in footer.
- CSV export link

### Bot Health
- Left: env var SET/MISSING table (no values), Railway deployment info
- Right: live log tail — last 100 stdlib log lines, color-coded by level, auto-scrolling

---

## FSM State Descriptions (Plain English)

These exact descriptions must appear in the UI when each state is active:

| State | Description |
|-------|-------------|
| IDLE | Bot is waiting for the next 15-minute window to open before evaluating. |
| EVALUATING | Bot is checking signals — computing edge and vol regime for this window. |
| ENTERING | Order placed. Waiting for fill confirmation from Kalshi. |
| POSITIONED | Trade is open. Bot is monitoring the position and waiting for the settlement window. |
| SETTLING | Within 60 seconds of window close. Waiting for Kalshi to publish the final result. |

---

## Implementation Reference

- **Beta bot:** `src/ops/dashboard.py` — canonical implementation for KXBTC15M bots
- **Pattern origin:** `kalshi-15m-bot/dashboard.py` — stdlib HTTPServer + inline SPA

---

## Agent Instructions

When building any new bot in this platform:

1. Create `src/ops/dashboard.py` using the canonical pattern above
2. Wire it into `main.py` after boot: `register_shared_state(...)` → `start_dashboard()`
3. Add `is_paused()` check at the top of the trading loop
4. Add `healthcheckPath = "/health"` to `railway.toml`
5. Never ship a bot without a dashboard. It is not optional.

A dashboard is as fundamental as structured logging. If you're adding a trading bot to this platform without a UI, stop and add it.
