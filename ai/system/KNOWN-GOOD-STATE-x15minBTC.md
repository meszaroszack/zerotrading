# KNOWN-GOOD-STATE — ZeroTrading x15minBTC

> **As of:** 2026-05-23 22:56 ET (commit `928c897` on `main`)
> **Memorialized by:** Perplexity Computer at operator's direction:
> *"lets gooo!! we are not going backwards from here - memorialize this"*

This document records the **first confirmed working configuration** of the
15-min BTC bot on Railway production. Any future agent or operator who
breaks something can `git diff 928c897` against the current tree to find
the regression.

---

## Layer-0 invariants (DO NOT REGRESS)

1. **BTC price feed = REST polling.**
   - Primary: `https://data-api.binance.vision/api/v3/klines`
   - Fallback: `https://api.exchange.coinbase.com/products/BTC-USD/candles`
   - 5-second poll loop.
   - Implementation: `src/ingestion/btc_feed.py` → `BTCRestFeed`.
   - **Never WebSocket.** Geo-block proven by CRASH-023, CRASH-026.
   - Architecture doc: `ai/system/MARKET-DATA-ARCHITECTURE.md`.

2. **Kalshi liveness = WS connection state, NOT message recency.**
   - `KalshiWebSocketClient.is_connected` (`self._ws is not None`).
   - 5-second heartbeat in `main.py:kalshi_heartbeat_task` calls
     `feed_health.record_kalshi_update()` while connected.
   - 15-min markets are sparse between windows. Empty orderbook ≠ dead WS.
   - CRASH-028.

3. **Status builders are pure reads.**
   - `_build_status()` and `FeedHealthMonitor.status_dict()` MUST NOT have
     logging side-effects. Use `is_healthy(log_when_stale=False)` from the
     status path; only the trading loop's gate uses the loud variant.
   - CRASH-028.

4. **Log buffer bridge.**
   - `src/ops/logging.py:_emit` mirrors every JSON line to
     `logging.getLogger("zerotrading")` with `propagate=True`, so the root
     `_LogBufferHandler` in `src/ops/health.py` captures it for the
     dashboard `/api/status.log_tail`.
   - Handler timestamp uses `datetime.fromtimestamp(record.created, tz=utc)`
     — **never** `self.formatTime` (lives on Formatter, not Handler).
   - CRASH-025, CRASH-027.

5. **Kalshi WS subscribe payload — three channels, one cmd each.**
   - `orderbook_delta`, `user_orders`, `fill`.
   - `params.market_ticker` (singular string) is valid; per-market filter.
   - Server returns `subscribed` ack with a `sid` per channel, then
     `orderbook_snapshot` immediately, then `orderbook_delta` incrementals.

6. **Window close time is server-computed.**
   - `_build_status()` derives from UTC clock alignment (xx:00/15/30/45),
     not from ticker parsing. Kalshi tickers embed ET-local time and are
     not parseable client-side.

---

## Verified `/api/status` shape (2026-05-23 22:56 ET)

```json
{
  "kalshi_ws_healthy": true,
  "binance_ws_healthy": true,
  "btc_spot": 76836.51,
  "active_ticker": "KXBTC15M-26MAY232300-00",
  "window_close_time": "2026-05-24T03:00:00Z",
  "fsm_state": "IDLE",
  "mode": "production",
  "log_tail": [ ... 62 entries, diverse categories ... ]
}
```

Dashboard pills (verified by operator screenshot `image-3.jpg`):
- Top bar: **Kalshi WS: LIVE 🟢** + **Binance WS: LIVE 🟢**.
- Footer: same.
- BTC SPOT card: $76,847.
- Window countdown: counting down correctly.
- Current ticker: `KXBTC15M-26MAY232300-00`.

Log evidence the WS path is healthy end-to-end:
```
kalshi_ws.connecting url=wss://api.elections.kalshi.com/trade-api/ws/v2
kalshi_ws.connected ticker=KXBTC15M-26MAY232300-00
kalshi_ws.subscribe_sent channel=orderbook_delta
kalshi_ws.subscribed sid=1 channel=orderbook_delta
kalshi_ws.ob_snapshot ticker=KXBTC15M-26MAY232300-00 yes_levels=138 no_levels=68
```

---

## Currently-known non-blockers (observed but acceptable in this snapshot)

- `loop.guardrail_blocked reason=safety floor breach: balance $17.28 < floor $50.00`
  — every loop tick. Bot is *correctly* refusing to trade because the
  Kalshi account balance is below `MIN_BALANCE_USD_FLOOR`. This is the
  guardrail working as designed. Resolve by funding the account OR
  lowering the floor — both operator decisions, not a code bug.
- `health.tick.error 'Query' object has no attribute 'gte'`
  — once per minute. Cosmetic Supabase/local-store query API mismatch in
  `health_loop`. Does not block trading. To be fixed in a follow-up.
- Dashboard "Waiting for orderbook data..." text in the Active Market
  Window card while `yes_levels=138, no_levels=68` exist in the bot's
  state. Wiring bug from `latest_ob` → dashboard mid-price widget.
  Bot itself sees the book.

---

## Git anchors

- `928c897` — PR #6 merge — current known-good HEAD.
- `f8e1a19` — PR #5 merge — REST feed swap (CRASH-026, CRASH-027).
- `02f6666` — PR #4 merge — dashboard countdown + log_tail bridge (CRASH-024, CRASH-025).
- `788512d` — PR #3 merge — Kalshi WS rewrite against real protocol (CRASH-017/018/019).

To audit a future regression:
```bash
git diff 928c897..HEAD -- main.py src/ingestion/ src/ops/
```

If the diff touches any of the invariants above without a CRASH-NNN entry
explaining why, that diff is the regression. Revert and ask the operator.

---

## Operator directive (verbatim)

> "lets gooo!! we are not going backwards from here - memorialize this -
> now it doesnt seem to be fully reading the trades if you look over on
> the left but that is okay for now"
> — 2026-05-23 22:56 ET

This document is that memorialization.
