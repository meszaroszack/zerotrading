# research/adapted/

Distilled, adapted patterns lifted from external repos, papers, or prior builds.
One file per source, describing what we are reusing and why.

## Filename convention
`<source>-<topic>.md`

Examples:
- `myrepo-15m-launch-wedge.md`
- `kalshi-public-docs-orderbook.md`
- `paper-mystic-original.md`

## Template
```
# <Source> - <Topic>

**Source URL:** <link>
**Commit / version:** <hash, tag, or date>
**License:** <MIT / Apache / unknown / proprietary-with-permission>
**Pulled by:** <agent / human> on YYYY-MM-DD

## What it does
<2-5 sentences.>

## What is reusable for ZeroTrading
- Pattern 1: ...
- Pattern 2: ...

## What is wrong / not portable
- Issue 1: ...

## Port plan
- Where in ZeroTrading this lands (file paths).
- What gets rewritten vs copied.
- DECISION-LOG entry: <link or `pending`>.

## Open questions
- ...
```

## Rules
- Cite sources. Respect licenses. Do not paste large copyrighted blocks.
- Every file here must have a matching DECISION-LOG entry (Status: proposed | decided | rejected).
- If adapted code lands in the repo, link from the code file back to this note.
