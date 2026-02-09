# Snapshot: 2026-02-09 Clean Root

Purpose: keep project root focused on active CC100x functional assets while preserving all historical material for future reference.

## Moved from Root

- `reference/` -> `archive/2026-02-09-clean-root/reference/`
- `docs/BUILD-PLAN.md` -> `archive/2026-02-09-clean-root/docs/BUILD-PLAN.md`
- `docs/cc100x-bible.md` -> `archive/2026-02-09-clean-root/docs/cc100x-bible.md`
- `docs/cc100x-orchestration-logic-analysis.md` -> `archive/2026-02-09-clean-root/docs/cc100x-orchestration-logic-analysis.md`
- `docs/audit/` -> `archive/2026-02-09-clean-root/docs/audit/`
- `docs/deep-audit-agent-team-parity-2026-02-09/` -> `archive/2026-02-09-clean-root/docs/deep-audit-agent-team-parity-2026-02-09/`
- `plan/` -> `archive/2026-02-09-clean-root/plan/`

## Restore Examples

```bash
# Restore one file
mv archive/2026-02-09-clean-root/docs/cc100x-bible.md docs/cc100x-bible.md

# Restore full legacy reference set
mv archive/2026-02-09-clean-root/reference reference
```

## Notes

- This snapshot is archival only.
- Current canonical governance docs are in `docs/functional-governance/`.
