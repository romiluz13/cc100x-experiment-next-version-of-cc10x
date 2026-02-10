# Functional Governance (CC100x)

This folder defines how documentation stays correct while CC100x evolves.

## Canonical Rule

Only these folders are functional source of truth:

- `plugins/cc100x/skills`
- `plugins/cc100x/agents`

Everything else is derived documentation.

## Files

- `protocol-manifest.md`
  Canonical orchestration map extracted from functional files.
- `cc100x-bible-functional.md`
  Functional bible with source citations for every rule.
- `improvement-skill-reference.md`
  Contract for future improvement work (what to read first, what to validate).

## Drift Check

Run:

```bash
npm run check:functional-bible
npm run check:protocol-integrity
npm run check:artifact-policy
```

This validates that bible citations point only to functional files and that cited files/lines exist.
It also validates that lead protocol gates and artifact-governance constraints remain intact.
