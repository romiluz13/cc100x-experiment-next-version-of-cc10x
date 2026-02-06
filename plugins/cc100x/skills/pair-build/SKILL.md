---
name: pair-build
description: "Real-time pair programming using Agent Teams. Builder implements with live reviewer feedback, then hunter and verifier run final checks."
---

# Pair Build Protocol

## Overview

Pair Build creates a real-time pair programming experience. The Builder implements code module by module, pausing after each to get feedback from the Live Reviewer. After implementation, the Hunter scans for silent failures and the Verifier runs E2E tests.

---

## Team Composition

| Teammate | Role | Mode | When Active |
|----------|------|------|-------------|
| **builder** | Implements code using TDD | READ+WRITE | Phase 1 (entire implementation) |
| **live-reviewer** | Reviews in real-time as builder works | READ-ONLY | Phase 1 (alongside builder) |
| **hunter** | Scans for silent failures | READ-ONLY | Phase 2 (after builder completes) |
| **verifier** | Runs E2E verification | READ-ONLY | Phase 3 (after hunter completes) |

**File ownership:** ONLY the builder edits files. All other teammates are READ-ONLY.

---

## Protocol Phases

### Phase 1: Implementation Loop (Builder + Live Reviewer)

**Builder and Live Reviewer start simultaneously.**

**The loop:**
```
Builder implements module 1
  → Builder messages live-reviewer: "Review src/auth/middleware.ts"
  → Live Reviewer reads file, messages back: "LGTM" or "STOP: [issue]"
  → If STOP: Builder fixes inline, messages reviewer again
  → If LGTM: Builder continues to module 2

Builder implements module 2
  → Same review loop

...

Builder messages live-reviewer: "Implementation complete"
  → Live Reviewer finishes their task
```

**Builder rules:**
- Follow TDD: RED → GREEN → REFACTOR for each module
- After each module, pause and request review
- Wait for reviewer response before continuing
- If reviewer says STOP → fix before continuing
- Own ALL file edits (no other teammate edits files)

**Live Reviewer rules:**
- READ-ONLY: never edit files
- When builder requests review:
  - Read the specific file(s)
  - Check: security, correctness, pattern adherence
  - Reply with: "LGTM" (continue) or "STOP: [critical issue]" (must fix)
  - Keep feedback focused (not a full review - save that for Review Arena)
- Only say STOP for actual blocking issues (security, correctness)
- Pattern suggestions are "NOTE: [suggestion]" (non-blocking)

### Phase 2: Silent Failure Hunt (Hunter)

**Starts after builder completes.**

Hunter scans the entire implementation for:
- Empty catch blocks
- Log-only error handlers
- Generic error messages ("Something went wrong")
- Swallowed exceptions
- Missing error boundaries

**Severity classification:**
| Severity | Definition | Blocks Ship? |
|----------|-----------|-------------|
| CRITICAL | Data loss, security, crash | YES |
| HIGH | Wrong behavior, degraded UX | Should fix |
| MEDIUM | Suboptimal but functional | Track as TODO |
| LOW | Code smell | Optional |

**If CRITICAL issues found:** Lead creates remediation task for builder.

### Phase 3: E2E Verification (Verifier)

**Starts after hunter completes.**

Verifier runs:
1. All unit tests
2. Integration tests (if available)
3. E2E scenarios
4. Build verification

Every scenario needs PASS/FAIL with exit code evidence.

**If verification fails:**
- Option A: Create fix task (fixable issue)
- Option B: Revert changes (fundamental design issue)
- Option C: Document limitation + get user approval

---

## Task Structure

```
CC100X BUILD: {feature}
├── CC100X builder: Implement {feature}
├── CC100X live-reviewer: Real-time review (starts with builder)
├── CC100X hunter: Silent failure audit (blocked by builder)
├── CC100X verifier: E2E verification (blocked by hunter)
└── CC100X Memory Update: Persist build learnings (blocked by verifier)
```

---

## Plan-First Gate

Before starting Pair Build, check:
1. Does a plan exist? (Check `activeContext.md ## References`)
2. If no plan → AskUserQuestion: "Plan first (Recommended) / Build directly"
3. If plan exists → builder receives plan file path in prompt

**Builder must read and follow the plan.** Plan is the source of truth for what to build.

---

## Remediation Loop

If hunter finds CRITICAL issues or verifier fails:

1. Lead creates `CC100X REM-FIX: {issue}` task
2. Builder fixes the issue
3. Abbreviated re-verification (affected scenarios only)
4. If passes → proceed to Memory Update
5. If fails again → escalate to user

---

## Memory Notes Collection

At workflow end, lead collects from all teammates:

| Teammate | Memory Notes |
|----------|-------------|
| Builder | What was built, TDD evidence, patterns used |
| Live Reviewer | Code quality observations, pattern adherence |
| Hunter | Error handling patterns, silent failure risks |
| Verifier | E2E results, integration observations |

All notes merged and persisted to `.claude/cc100x/` files.
