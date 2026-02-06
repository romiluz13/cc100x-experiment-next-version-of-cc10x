---
name: router-contract
description: |
  Defines the YAML Router Contract format that ALL CC100x teammates must output.
  Used by the lead to validate teammate work and make orchestration decisions.
---

# Router Contract (CC100x)

## Purpose

The Router Contract is a machine-readable YAML block output by every teammate at the end of their work. The lead uses it to:
- Validate teammate output quality
- Decide whether to proceed or block
- Collect memory notes for persistence
- Resolve conflicts between teammates

---

## Contract Format (REQUIRED)

Every teammate MUST include this section at the end of their output:

```markdown
### Router Contract (MACHINE-READABLE)
```yaml
STATUS: [value from agent-specific list below]
CONFIDENCE: [0-100]
CRITICAL_ISSUES: [count]
BLOCKING: [true|false]
REQUIRES_REMEDIATION: [true|false]
REMEDIATION_REASON: [null or description of what needs fixing]
MEMORY_NOTES:
  learnings: ["insight 1", "insight 2"]
  patterns: ["pattern or gotcha discovered"]
  verification: ["evidence of what was verified"]
```
```

---

## STATUS Values by Agent

| Agent | Valid STATUS Values | Description |
|-------|-------------------|-------------|
| **builder** | `PASS`, `FAIL` | PASS = TDD evidence present (RED exit=1, GREEN exit=0) |
| **security-reviewer** | `APPROVE`, `CHANGES_REQUESTED` | APPROVE requires CRITICAL_ISSUES=0 and CONFIDENCE>=80 |
| **performance-reviewer** | `APPROVE`, `CHANGES_REQUESTED` | APPROVE requires CRITICAL_ISSUES=0 and CONFIDENCE>=80 |
| **quality-reviewer** | `APPROVE`, `CHANGES_REQUESTED` | APPROVE requires CRITICAL_ISSUES=0 and CONFIDENCE>=80 |
| **live-reviewer** | `APPROVE`, `CHANGES_REQUESTED` | APPROVE requires no STOP-level issues raised |
| **hunter** | `CLEAN`, `ISSUES_FOUND` | CLEAN requires CRITICAL_ISSUES=0 |
| **verifier** | `PASS`, `FAIL` | PASS requires all scenarios passed |
| **investigator** | `FIXED`, `INVESTIGATING`, `BLOCKED` | FIXED requires TDD evidence + variant coverage |
| **planner** | `PLAN_CREATED`, `NEEDS_CLARIFICATION` | PLAN_CREATED requires plan file saved + CONFIDENCE>=5 |

---

## Field Definitions

### STATUS
Agent's self-reported completion status. See table above for valid values per agent.

### CONFIDENCE
0-100 score indicating how confident the agent is in their output.
- Reviewers: >=80 to report findings
- Builder: based on assumption certainty
- Investigator: based on evidence strength
- Planner: 1-10 scale (mapped to 10-100)

### CRITICAL_ISSUES
Count of blocking issues found. Only reviewers and hunter use this.
- CRITICAL_ISSUES > 0 typically means BLOCKING=true

### BLOCKING
Whether the workflow should stop before proceeding to next agent.
- `true`: Lead must create remediation task before continuing
- `false`: Proceed to next agent/phase

### REQUIRES_REMEDIATION
Whether a fix task needs to be created.
- `true`: Create a `CC100X REM-FIX:` task with REMEDIATION_REASON as description
- `false`: No fix needed

### REMEDIATION_REASON
Exact text for the remediation task description. Null if no remediation needed.

### MEMORY_NOTES
Structured notes for the lead to persist to memory files at workflow end.
- **learnings**: Insights for `activeContext.md ## Learnings`
- **patterns**: Conventions or gotchas for `patterns.md ## Common Gotchas`
- **verification**: Evidence for `progress.md ## Verification`

---

## Contract Rules (Per Agent)

### Builder
```yaml
STATUS: PASS | FAIL
# CONTRACT RULE: STATUS=PASS requires TDD_RED_EXIT=1 AND TDD_GREEN_EXIT=0
TDD_RED_EXIT: [1 if red phase ran, null if missing]
TDD_GREEN_EXIT: [0 if green phase ran, null if missing]
```

### Reviewers (Security, Performance, Quality)
```yaml
STATUS: APPROVE | CHANGES_REQUESTED
# CONTRACT RULE: STATUS=APPROVE requires CRITICAL_ISSUES=0 and CONFIDENCE>=80
CRITICAL_ISSUES: [count]
HIGH_ISSUES: [count]
```

### Hunter
```yaml
STATUS: CLEAN | ISSUES_FOUND
# CONTRACT RULE: STATUS=CLEAN requires CRITICAL_ISSUES=0
CRITICAL_ISSUES: [count]
HIGH_ISSUES: [count]
```

### Verifier
```yaml
STATUS: PASS | FAIL
# CONTRACT RULE: STATUS=PASS requires BLOCKERS=0 and SCENARIOS_PASSED=SCENARIOS_TOTAL
SCENARIOS_TOTAL: [count]
SCENARIOS_PASSED: [count]
BLOCKERS: [count]
```

### Investigator
```yaml
STATUS: FIXED | INVESTIGATING | BLOCKED
# CONTRACT RULE: STATUS=FIXED requires TDD_RED_EXIT=1 AND TDD_GREEN_EXIT=0 AND VARIANTS_COVERED>=1
ROOT_CAUSE: "[one-line summary]"
TDD_RED_EXIT: [1 or null]
TDD_GREEN_EXIT: [0 or null]
VARIANTS_COVERED: [count]
```

### Planner
```yaml
STATUS: PLAN_CREATED | NEEDS_CLARIFICATION
# CONTRACT RULE: STATUS=PLAN_CREATED requires PLAN_FILE is valid path and CONFIDENCE>=5
PLAN_FILE: "[path]"
PHASES: [count]
CONFIDENCE: [1-10]
```

---

## Lead Validation Logic

### Step 1: Check Contract Exists
```
Look for "### Router Contract (MACHINE-READABLE)" in agent output.
If NOT found → non-compliant output. Block downstream. Request re-output.
```

### Step 2: Validate Contract Fields
```
Parse YAML block.

Circuit Breaker (BEFORE creating any REM-FIX):
- If 3+ REM-FIX tasks already exist → AskUserQuestion:
  - Research best practices (Recommended)
  - Fix locally
  - Skip (not recommended)
  - Abort

Rules:
1. If contract.BLOCKING == true OR contract.REQUIRES_REMEDIATION == true:
   → Create CC100X REM-FIX task
   → Block downstream tasks
   → STOP

2. If contract.CRITICAL_ISSUES > 0 AND parallel phase (multiple reviewers):
   → Check for conflicts between reviewers
   → If security says CRITICAL but others disagree → security wins
   → AskUserQuestion if genuine conflict

3. Collect contract.MEMORY_NOTES for persistence

4. If none triggered → Proceed to next phase
```

### Step 3: Output Validation Evidence
```markdown
### Agent Validation: {agent_name}
- Router Contract: Found
- STATUS: {value}
- BLOCKING: {value}
- CRITICAL_ISSUES: {value}
- Proceeding: [Yes/No + reason]
```

---

## Remediation Loop

When a REM-FIX task is created (code changes needed):

1. Fix is implemented
2. Re-review is triggered (abbreviated Review Arena)
3. Only after re-review passes → proceed to next phase

**This is non-negotiable.** Code changes without re-review break orchestration integrity.
