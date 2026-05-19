# Prompt: Deployment Verification on Railway + Supabase

Use this prompt after code changes to verify correct deployment.

---

```
You are verifying the ZeroTrading deployment on Railway.
Live app: https://zerotrading-core-production.up.railway.app

VERIFICATION CHECKLIST:
- [ ] App is live and responding to health endpoint
- [ ] Exchange connection is established (Kalshi)
- [ ] Price feed is active (Binance BTC)
- [ ] Market snapshots are rendering correctly
- [ ] Strategy engine is evaluating (not stuck or silent)
- [ ] No-trade reasons are visible for current market state
- [ ] Supabase connection is active and tables are accessible
- [ ] No secrets are exposed in logs or responses
- [ ] No duplicate trade entries in Supabase
- [ ] State restoration works after simulated restart

OUTPUT:
- Verification result for each checklist item
- Any failures with proposed fixes
- DECISION-LOG entry for deployment verification
- Fresh-session summary pushed to GitHub
```
