# Railway Deployment Runbook

## Guidelines

This runbook is the **traditional ops doc** for deploying and operating ZeroTrading on Railway with Supabase as the persistence/analytics layer. It is not exhaustive yet — fill in environment-specific details as the deployment matures.

Audience: operators (human) running deployments, restarts, rollbacks, or incident response.

Non-negotiables (echo from `ai/system/REPO-PRINCIPLES.md`):
- Never commit secrets to the repo.
- Exchange truth wins over local state when reconciling after a deploy.
- Fail closed on ambiguity; if a deploy is in an unknown state, do not enable live trading.

## Stack

- **App platform:** Railway
- **Persistence / analytics:** Supabase (Postgres)
- **Source:** GitHub `meszaroszack/zerotrading`
- **Live app (current):** https://zerotrading-core-production.up.railway.app

## Environments

| Environment | Purpose | Branch / source | Mode |
|---|---|---|---|
| `production` | Live operator-facing app | `main` | safe-live / live |
| `staging` | Pre-prod verification | feature branches | paper / safe-live |
| `local` | Developer machine | working copy | paper |

If the Railway project shows a failing environment (e.g. `dazzling-mindfulness/production`), investigate before merging more changes.

## Required environment variables

These are stored in Railway's environment, never in the repo. Document them here by name only.

```
KALSHI_API_KEY_ID         # Kalshi API key identifier
KALSHI_API_PRIVATE_KEY    # Kalshi RSA private key (PEM)
KALSHI_ENV                # 'demo' | 'production'
SUPABASE_URL              # Supabase project URL
SUPABASE_ANON_KEY         # Supabase anon key (frontend)
SUPABASE_SERVICE_ROLE_KEY # Supabase service role (server-only)
BINANCE_OR_COINBASE_WS_URL # BTC price feed websocket
APP_MODE                  # 'paper' | 'safe-live' | 'live' | 'docs-only'
LOG_LEVEL                 # 'debug' | 'info' | 'warn' | 'error'
DAILY_LOSS_CAP_USD
MAX_CONCURRENT_POSITIONS
SAFETY_FLOOR_USD
```

Review this list with every PR that adds a new env var.

## Deploy procedure (normal)

1. PR is opened and reviewed against `main`.
2. CI passes (lint, tests, evals).
3. Merge to `main`.
4. Railway auto-deploys `main` to `production`.
5. Watch the deploy logs for:
   - successful build
   - successful migration (if Supabase schema changed)
   - successful Kalshi auth handshake
   - successful price feed subscription
   - successful position reconciliation against exchange
6. Verify the dashboard shows healthy heartbeat, latency, and feed status.
7. Append a deploy entry to `ai/summaries/DECISION-LOG.md` if anything notable happened.

## Deploy procedure (safe-live first)

When pushing a change that affects trading behavior:

1. Set `APP_MODE=safe-live` in the Railway environment.
2. Set `MAX_CONCURRENT_POSITIONS=1`.
3. Set sizing to tiny.
4. Watch one or two full trade lifecycles end to end.
5. Verify reconciliation against Kalshi positions.
6. Only then flip `APP_MODE` to `live` and raise caps.

## Rollback

If a deploy is bad:

1. In Railway, redeploy the last known-good commit.
2. Confirm the dashboard shows healthy state and reconciled positions.
3. Open an incident entry in `ai/summaries/DECISION-LOG.md` with:
   - what broke
   - how it was detected
   - what was reverted
   - what follow-up fix is needed

## Database / Supabase

- Schema migrations live in `supabase/migrations/` (to be created when first migration lands).
- Never run a destructive migration in production without:
  - a backup,
  - operator approval,
  - a documented rollback path.
- Service role keys are server-only and must never appear in client code or browser bundles.

## Monitoring during operation

At minimum, verify these regularly:

- Heartbeat / uptime
- WebSocket latency
- Kalshi auth status
- Price feed status
- Active positions vs exchange truth
- Balance vs exchange truth
- Strategy decision events (trades + skips)
- Reconciliation / correction events
- Daily loss vs cap

## Incident response

1. Stop new entries: set `APP_MODE=docs-only` or `paper` and redeploy.
2. Reconcile open positions against Kalshi truth.
3. Capture logs and a snapshot of dashboard state.
4. Open a DECISION-LOG entry with the incident summary.
5. Do not resume `live` mode until the root cause is identified.

## TODO for this runbook

- [ ] Document Supabase project IDs and per-env URLs (without keys).
- [ ] Add concrete dashboard health-check screenshots.
- [ ] Document Railway custom domain and SSL setup.
- [ ] Document log retention and alerting setup.
