# Improvement Skill Reference Contract

This is the operating contract for any future "improvement" workflow that updates CC100x orchestration.

## Read Order (Mandatory)

1. `docs/functional-governance/protocol-manifest.md`
2. `docs/functional-governance/cc100x-bible-functional.md`
3. Functional targets only:
   - `plugins/cc100x/skills`
   - `plugins/cc100x/agents`

## Hard Rules

- Do not treat README/docs/reference as runtime truth.
- Do not add bible statements without a functional citation.
- Do not change workflow chains without updating manifest + functional bible together.
- Do not merge if drift check fails.

## Required Validation

Run before commit/PR:

```bash
npm run check:functional-bible
```

## Merge Checklist

- [ ] Functional change implemented in `plugins/cc100x/skills` or `plugins/cc100x/agents`
- [ ] `protocol-manifest.md` updated if behavior changed
- [ ] `cc100x-bible-functional.md` updated with citations
- [ ] Drift check passes
