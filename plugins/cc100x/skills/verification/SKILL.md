---
name: verification
description: "Internal skill. Use cc100x-lead for all development tasks."
allowed-tools: Read, Grep, Glob, Bash, LSP
---

# Verification Before Completion

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. REFLECT: Pause to consider tool results before next action
6. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Requirements met | Line-by-line checklist | Tests passing |

## Red Flags - STOP

If you find yourself:

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification
- About to commit/push/PR without verification
- Trusting agent success reports
- Relying on partial verification
- **ANY wording implying success without having run verification**

**STOP. Run verification. Get evidence. THEN speak.**

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Agent said success" | Verify independently |
| "The code looks correct" | Looking ≠ running |

## Self-Critique Gate (BEFORE Verification)

### Code Quality
- [ ] Follows patterns from reference files?
- [ ] Naming matches project conventions?
- [ ] Error handling in place?
- [ ] No debug artifacts?
- [ ] No hardcoded values that should be constants?

### Implementation Completeness
- [ ] All required files modified?
- [ ] Requirements fully met?
- [ ] No scope creep?

## Validation Levels

| Level | Name | Commands | When to Use |
|-------|------|----------|-------------|
| 1 | Syntax & Style | `npm run lint`, `tsc --noEmit` | Every task |
| 2 | Unit Tests | `npm test` | Low-Medium risk |
| 3 | Integration Tests | `npm run test:integration` | Medium-High risk |
| 4 | Manual Validation | User flow walkthrough | High-Critical risk |

## Goal-Backward Lens

After standard verification passes:

### Three Questions
1. **Truths:** What must be OBSERVABLE? (user-facing behaviors)
2. **Artifacts:** What must EXIST? (files, endpoints, tests)
3. **Wiring:** What must be CONNECTED? (component → API → database)

### Why This Catches Stubs
A component can:
- Exist ✓
- Pass lint ✓
- Have tests ✓
- But NOT be wired to the system ✗

Goal-backward asks: "Does the GOAL work?" not "Did the TASK complete?"

## Stub Detection Patterns

### Universal Stubs
```bash
grep -rE "TODO|FIXME|placeholder|not implemented|coming soon" --include="*.ts" --include="*.tsx"
grep -rE "return null|return undefined|return \{\}|return \[\]" --include="*.ts" --include="*.tsx"
```

### Wiring Verification
```bash
# Component → API
grep -E "fetch\(['\"].*api|axios\.(get|post)" src/components/
# API → Database
grep -E "prisma\.|db\.|mongoose\." src/app/api/
```

**If any stub patterns found:** DO NOT claim completion. Fix or document why intentional.

## Output Format

```markdown
## Verification Summary

### Scope
[What was completed]

### Evidence

| Check | Command | Exit Code | Result |
|-------|---------|-----------|--------|
| Tests | `npm test` | 0 | PASS (34/34) |
| Build | `npm run build` | 0 | PASS |

### Status
COMPLETE - All verifications passed with fresh evidence
```

## The Bottom Line

**No shortcuts for verification.**

Run the command. Read the output. THEN claim the result.

This is non-negotiable.
