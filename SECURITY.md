# Security Policy

## Reporting Vulnerabilities

Do not open public GitHub issues for security vulnerabilities. Email the maintainer directly or open a private advisory.

## Secret Management

- All secrets, API keys, and credentials must be stored in Railway environment config or Supabase secrets.
- Never commit secrets to the repo, including in test files, docs, or AI session summaries.
- Rotate any key immediately if it is accidentally committed. Remove it from Git history.

## Sensitive Information

The following must never appear in the public repo:
- API keys or tokens
- Live account identifiers or balances
- Private infrastructure topology
- Unsafe admin/control endpoints

## Operational Security Notes

This project is open-source by design, but that does not mean operationally unsafe. The algorithm and research are public. The production secrets and live account details are not.

See `ai/system/OPEN-OPSEC-POLICY.md` for the full policy.
