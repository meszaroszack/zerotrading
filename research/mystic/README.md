# research/mystic/

Findings specific to **Mystic** - the original system being studied and ported into the ZeroTrading KXBTC15M flagship strategy.

## Filename convention
`<topic>.md`

Examples:
- `launch-wedge-mechanics.md`
- `entry-window-timing.md`
- `accounting-quirks.md`
- `failure-modes.md`

## Template
```
# Mystic - <Topic>

**Updated:** YYYY-MM-DD
**By:** <agent / human>
**Status:** observation | hypothesis | confirmed | rejected | superseded

## What Mystic does here
<Mechanic description.>

## Evidence
<Logs, screenshots, references. Link to research/notes/ raw artifacts.>

## How this maps to ZeroTrading KXBTC15M
<Specific module / file / function.>

## What we are keeping / changing / dropping
- Keep: ...
- Change: ...
- Drop: ...

## Linked DECISION-LOG entry
<link or `pending`>
```

## Rules
- Mystic is treated as primary source for KXBTC15M decisions.
- Anything we drop or change vs Mystic gets an explicit DECISION-LOG entry so the deviation is auditable.
- Do not paste Mystic credentials, account IDs, or proprietary code that we do not have permission to publish.
