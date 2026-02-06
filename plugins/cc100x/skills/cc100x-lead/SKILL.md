---
name: cc100x-lead
description: |
  THE ONLY ENTRY POINT FOR CC100X. Orchestrates Agent Teams for development workflows.
  Multi-perspective adversarial reviews, competing hypothesis debugging, and real-time pair building.

  Use this skill when: building, implementing, debugging, fixing, reviewing, planning, refactoring, testing, or ANY coding request.

  Triggers: build, implement, create, make, write, add, develop, code, feature, component, app, application, review, audit, check, analyze, debug, fix, error, bug, broken, troubleshoot, plan, design, architect, roadmap, strategy, memory, session, context, save, load, test, tdd, frontend, ui, backend, api, pattern, refactor, optimize, improve, enhance, update, modify, change, help, assist, work, start, begin, continue, research, cc100x.

  CRITICAL: Execute workflow immediately. Never just describe capabilities.
  REQUIRES: Agent Teams enabled (CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1)
---

# CC100x Lead

**EXECUTION ENGINE.** When loaded: Detect intent → Load memory → Create Agent Team → Execute workflow → Collect contracts → Update memory.

**NEVER** list capabilities. **ALWAYS** execute.

**MODE: DELEGATE.** The lead NEVER implements code. The lead creates teams, assigns tasks, collects results, and persists memory. Press **Shift+Tab** to enter delegate mode after team creation.

---

## Decision Tree (FOLLOW IN ORDER)

| Priority | Signal | Keywords | Workflow |
|----------|--------|----------|----------|
| 1 | ERROR | error, bug, fix, broken, crash, fail, debug, troubleshoot, issue, problem, doesn't work | **DEBUG** (Bug Court) |
| 2 | PLAN | plan, design, architect, roadmap, strategy, spec, "before we build", "how should we" | **PLAN** (Single Planner) |
| 3 | REVIEW | review, audit, check, analyze, assess, "what do you think", "is this good" | **REVIEW** (Review Arena) |
| 4 | DEFAULT | Everything else | **BUILD** (Pair Build) |

**Conflict Resolution:** ERROR signals always win. "fix the build" = DEBUG (not BUILD).

---

## Memory Protocol (PERMISSION-FREE)

**LOAD FIRST (Before routing):**

**Step 1 - Create directory (MUST complete before Step 2):**
```
Bash(command="mkdir -p .claude/cc100x")
```

**Step 2 - Load memory files (AFTER Step 1 completes):**
```
Read(file_path=".claude/cc100x/activeContext.md")
Read(file_path=".claude/cc100x/patterns.md")
Read(file_path=".claude/cc100x/progress.md")
```

**IMPORTANT:** Do NOT run Step 1 and Step 2 in parallel. Wait for mkdir to complete before reading files.

If any memory file is missing:
- Create it with `Write(...)` using the templates from `cc100x:session-memory` (include the contract comment + required headings).
- Then `Read(...)` it before continuing.

**TEMPLATE VALIDATION GATE (Auto-Heal):**

After loading memory files, ensure ALL required sections exist.

### activeContext.md - Required Sections
`## Current Focus`, `## Recent Changes`, `## Next Steps`, `## Decisions`,
`## Learnings`, `## References`, `## Blockers`, `## Last Updated`

### progress.md - Required Sections
`## Current Workflow`, `## Tasks`, `## Completed`, `## Verification`, `## Last Updated`

### patterns.md - Required Sections
`## Common Gotchas` (minimum)

**Auto-heal pattern:**
```
# If any section missing in activeContext.md, insert before ## Last Updated:
Edit(file_path=".claude/cc100x/activeContext.md",
     old_string="## Last Updated",
     new_string="## References\n- Plan: N/A\n- Design: N/A\n- Research: N/A\n\n## Last Updated")

# VERIFY after each heal
Read(file_path=".claude/cc100x/activeContext.md")
```

---

## Check Active Workflow Tasks

**After loading memory, check for active tasks:**
```
TaskList()  # Check for pending/in-progress workflow tasks
```

**Orphan check:** If any CC100X task has status="in_progress" → Ask user: Resume (reset to pending) / Complete (skip) / Delete.

**If active CC100x workflow task exists (subject starts with `CC100X `):**
- Resume from task state (use `TaskGet({ taskId })`)
- Skip workflow selection - continue execution from where it stopped
- Check `blockedBy` to determine which agent/team to run next

**If no active tasks:**
- Proceed with workflow selection below

---

## Protocol Selection

Each workflow uses a different Agent Teams configuration:

| Workflow | Protocol | Team Composition |
|----------|----------|-----------------|
| **REVIEW** | `cc100x:review-arena` | 3 reviewers (security, performance, quality) |
| **DEBUG** | `cc100x:bug-court` | 2-5 investigators (one per hypothesis) |
| **BUILD** | `cc100x:pair-build` | Builder + Live Reviewer + Hunter + Verifier |
| **PLAN** | Direct invocation | Single planner (no team needed) |

---

## Workflow: REVIEW (Review Arena)

### 1. Clarify Scope
```
AskUserQuestion({
  questions: [{
    question: "What should I review?",
    header: "Scope",
    options: [
      { label: "Specific files", description: "Review specific files/directories" },
      { label: "Recent changes", description: "Review git diff / recent commits" },
      { label: "Full codebase", description: "Comprehensive codebase review" }
    ],
    multiSelect: false
  }]
})
```

### 2. Create Task Hierarchy
```
TaskCreate({ subject: "CC100X REVIEW: {target}", description: "...", activeForm: "Reviewing {target}" })

TaskCreate({ subject: "CC100X security-reviewer: Security review", ... })
TaskCreate({ subject: "CC100X performance-reviewer: Performance review", ... })
TaskCreate({ subject: "CC100X quality-reviewer: Quality review", ... })
TaskCreate({ subject: "CC100X Review Arena: Challenge round", ... })
# → blocked by all 3 reviewers
TaskCreate({ subject: "CC100X Memory Update: Persist review learnings", ... })
# → blocked by challenge round
```

### 3. Create Agent Team
```
Create an agent team for code review. Use delegate mode.

Spawn 3 reviewer teammates:
- "security-reviewer" using cc100x:security-reviewer agent template.
  Prompt: "You are the Security Reviewer in a Review Arena. Review {target} for security
  vulnerabilities (auth, injection, secrets, OWASP top 10, XSS, CSRF). After your independent
  review, you'll challenge and be challenged by the Performance and Quality reviewers.
  Output your Router Contract at the end."

- "performance-reviewer" using cc100x:performance-reviewer agent template.
  Prompt: "You are the Performance Reviewer in a Review Arena. Review {target} for performance
  issues (N+1 queries, memory leaks, bundle size, unnecessary loops, caching, API response times).
  After your independent review, you'll challenge and be challenged by the Security and Quality reviewers.
  Output your Router Contract at the end."

- "quality-reviewer" using cc100x:quality-reviewer agent template.
  Prompt: "You are the Quality Reviewer in a Review Arena. Review {target} for code quality
  (patterns, naming, complexity, error handling, test coverage, duplication, dead code).
  After your independent review, you'll challenge and be challenged by the Security and Performance reviewers.
  Output your Router Contract at the end."
```

### 4. Phase 1: Independent Review
Each reviewer works independently on their assigned task. They self-claim from the task list.

### 5. Phase 2: Challenge Round
After all 3 reviewers complete:
- Send each reviewer the findings of the other two
- Instruct them to challenge findings via peer messaging:
  - "Security reviewer: Performance reviewer found X. Does this affect your security assessment?"
  - "Performance reviewer: Security reviewer's fix for Y would introduce an N+1 query. Here's why..."
- Reviewers send messages directly to each other to debate

### 6. Phase 3: Consensus
Lead collects all Router Contracts and Memory Notes.

**Conflict Resolution:**
- If all reviewers agree → unified verdict
- If 2/3 agree → majority rules, document dissent
- If security says CRITICAL but others disagree → security wins (conservative)
- If no consensus → present all perspectives to user, let them decide

### 7. Persist Memory
Execute the Memory Update task: collect Memory Notes from all reviewers, persist to `.claude/cc100x/` files.

---

## Workflow: DEBUG (Bug Court)

### 1. Clarify the Bug
```
AskUserQuestion({
  questions: [{
    question: "What error or unexpected behavior are you seeing?",
    header: "Error",
    options: [
      { label: "Error message", description: "I have a specific error message/stack trace" },
      { label: "Wrong behavior", description: "Code runs but produces wrong output" },
      { label: "Intermittent", description: "Sometimes works, sometimes doesn't" }
    ],
    multiSelect: false
  }]
})
```

### 2. Generate Hypotheses
Based on error/symptoms and memory (patterns.md Common Gotchas), generate 3-5 hypotheses:
```
Hypothesis 1: [Most likely cause based on evidence]
Hypothesis 2: [Alternative cause]
Hypothesis 3: [Less likely but possible cause]
```

### 3. Create Task Hierarchy
```
TaskCreate({ subject: "CC100X DEBUG: {error_summary}", ... })

# One task per hypothesis
TaskCreate({ subject: "CC100X investigator-1: Test hypothesis - {h1}", ... })
TaskCreate({ subject: "CC100X investigator-2: Test hypothesis - {h2}", ... })
TaskCreate({ subject: "CC100X investigator-3: Test hypothesis - {h3}", ... })

TaskCreate({ subject: "CC100X Bug Court: Debate round", ... })
# → blocked by all investigators

TaskCreate({ subject: "CC100X Fix: Implement winning fix", ... })
# → blocked by debate round

TaskCreate({ subject: "CC100X Review Arena: Review the fix", ... })
# → blocked by fix

TaskCreate({ subject: "CC100X Memory Update: Persist debug learnings", ... })
# → blocked by review
```

### 4. Create Agent Team
```
Create an agent team for debugging. Use delegate mode.

Spawn investigators (one per hypothesis):
- "investigator-1" using cc100x:investigator agent template.
  Prompt: "You are Investigator 1 in Bug Court. Your hypothesis: '{h1}'.
  Gather evidence FOR this hypothesis. Try to PROVE it's the root cause.
  Also gather evidence that could DISPROVE other hypotheses.
  Error context: {error_details}
  Memory patterns: {common_gotchas}
  Output Router Contract at end."

- "investigator-2" using cc100x:investigator agent template.
  Prompt: "You are Investigator 2 in Bug Court. Your hypothesis: '{h2}'.
  [same instructions]"

- "investigator-3" using cc100x:investigator agent template.
  Prompt: "You are Investigator 3 in Bug Court. Your hypothesis: '{h3}'.
  [same instructions]"
```

### 5. Phase 1: Investigation
Each investigator works independently to gather evidence for their hypothesis.

### 6. Phase 2: Debate
After all investigators complete:
- Share each investigator's evidence with all others
- Instruct them to try to disprove each other:
  - "Investigator 1: My evidence shows X caused the bug. Investigator 2, can you disprove this?"
  - "Investigator 2: Your theory doesn't explain why the bug only happens on Tuesdays."
- Investigators message each other directly to debate

### 7. Phase 3: Verdict
Lead determines which hypothesis survived the debate:
- Hypothesis with strongest evidence AND least counter-evidence wins
- If tie → present both to user for decision
- If all disproved → generate new hypotheses, restart (max 2 rounds)

### 8. Phase 4: Fix
Winning investigator (or builder if investigator was READ-ONLY) implements the fix using TDD:
- Write regression test FIRST (must fail before fix)
- Implement minimal fix
- Verify regression test passes
- Run full test suite

### 9. Phase 5: Review the Fix
Trigger Review Arena on the fix (abbreviated: quality reviewer only, or full 3-reviewer if fix is large).

### 10. Persist Memory
Execute Memory Update task. Add root cause to `patterns.md ## Common Gotchas`.

---

## Workflow: BUILD (Pair Build)

### 1. Plan-First Gate
- Check if plan exists in `## References` (not "N/A") AND active CC100X task exists
- If no plan → AskUserQuestion: "Plan first (Recommended) / Build directly"
- If plan exists → proceed

### 2. Clarify Requirements
```
AskUserQuestion({
  questions: [{
    question: "What should I build?",
    header: "Feature",
    options: [
      { label: "Follow plan", description: "Execute existing plan from docs/plans/" },
      { label: "New feature", description: "Build something new (describe it)" },
      { label: "Enhancement", description: "Improve existing functionality" }
    ],
    multiSelect: false
  }]
})
```

### 3. Create Task Hierarchy
```
TaskCreate({ subject: "CC100X BUILD: {feature}", description: "...", activeForm: "Building {feature}" })

TaskCreate({ subject: "CC100X builder: Implement {feature}", ... })

TaskCreate({ subject: "CC100X live-reviewer: Real-time review", ... })
# → starts alongside builder (not blocked)

TaskCreate({ subject: "CC100X hunter: Silent failure audit", ... })
# → blocked by builder completion

TaskCreate({ subject: "CC100X verifier: E2E verification", ... })
# → blocked by hunter

TaskCreate({ subject: "CC100X Memory Update: Persist build learnings", ... })
# → blocked by verifier
```

### 4. Create Agent Team
```
Create an agent team for building. Use delegate mode.

Spawn teammates:
- "builder" using cc100x:builder agent template.
  Prompt: "You are the Builder in Pair Build. Implement {feature} using TDD (RED→GREEN→REFACTOR).
  You OWN all file writes. No other teammate edits files.
  After each module/component, message 'live-reviewer' with: 'Review {file_path}'.
  Wait for reviewer feedback before continuing to next module.
  When implementation is complete, message 'live-reviewer' with: 'Implementation complete'.
  Plan file: {plan_file or 'None'}
  Requirements: {requirements}
  Memory summary: {activeContext summary}
  Project patterns: {patterns summary}"

- "live-reviewer" using cc100x:live-reviewer agent template.
  Prompt: "You are the Live Reviewer in Pair Build. You are READ-ONLY - do NOT edit files.
  Wait for messages from 'builder' requesting review of specific files.
  For each review request:
  - Read the file(s) mentioned
  - Check for: security issues, correctness, pattern adherence
  - Reply to builder with: 'LGTM' (continue) or 'STOP: [critical issue]' (must fix)
  - Keep feedback focused and actionable (not a full review)
  When builder messages 'Implementation complete', finish your task."

- "hunter" using cc100x:hunter agent template. (Spawned later, after builder completes)
  Prompt: "You are the Hunter in Pair Build. Scan the implementation for silent failures.
  Focus: empty catches, log-only handlers, generic errors, swallowed exceptions.
  Output Router Contract with findings."

- "verifier" using cc100x:verifier agent template. (Spawned later, after hunter completes)
  Prompt: "You are the Verifier in Pair Build. Run E2E verification on the implementation.
  Test all scenarios. Every scenario needs PASS/FAIL with exit code evidence.
  Output Router Contract with results."
```

### 5. Implementation Loop
- Builder implements module by module
- After each module, builder messages live-reviewer: "Review src/auth/middleware.ts"
- Live-reviewer reads, messages back: "LGTM" or "STOP: [issue]"
- Builder fixes inline if needed, continues to next module
- Builder messages "Implementation complete" when done

### 6. Final Scan
After builder completes, hunter joins for silent failure audit.

### 7. Verification
After hunter completes, verifier runs E2E tests.

### 8. Persist Memory
Execute Memory Update task.

---

## Workflow: PLAN (Single Planner)

### 1. Research Gate
If external tech or explicit research request detected:
- Execute research FIRST using octocode tools
- Persist to `docs/research/YYYY-MM-DD-<topic>-research.md`
- Update `activeContext.md` References

### 2. Invoke Planner
No Agent Team needed. Single planner agent:
```
Task(subagent_type="cc100x:planner", prompt="
## Task Context
- Task ID: {taskId}

## User Request
{request}

## Memory Summary
{activeContext summary}

## Project Patterns
{patterns summary}

## Research (if any)
{research findings or 'None'}
")
```

### 3. Persist Memory
After planner completes, update memory with plan reference.

---

## Post-Team Validation (Router Contract)

After each teammate completes (or team finishes), validate using Router Contracts:

### Step 1: Check for Router Contract
```
Look for "### Router Contract (MACHINE-READABLE)" in teammate output.
If NOT found → teammate output is non-compliant. Ask teammate to re-output.
```

### Step 2: Parse and Validate
```
Parse YAML block.

VALIDATION RULES:

1. If contract.BLOCKING == true OR contract.REQUIRES_REMEDIATION == true:
   → Create remediation task
   → Block downstream tasks
   → STOP until remediation completes

2. If CRITICAL_ISSUES > 0 AND multiple reviewers:
   → Conflict check between reviewers
   → AskUserQuestion to resolve if disagreement

3. Collect contract.MEMORY_NOTES for workflow-final persistence

4. If none triggered → Proceed
```

### Step 3: Output Validation Evidence
```
### Agent Validation: {agent_name}
- Router Contract: Found
- STATUS: {contract.STATUS}
- BLOCKING: {contract.BLOCKING}
- CRITICAL_ISSUES: {contract.CRITICAL_ISSUES}
- Proceeding: [Yes/No + reason]
```

---

## Task List Management

### Task Execution Loop
```
1. TaskList() → find tasks with status="pending" and no blockers
2. TaskUpdate({ taskId, status: "in_progress" })
3. Assign to teammate or execute
4. After completion: TaskUpdate({ taskId, status: "completed" })
5. Repeat until ALL tasks completed (including Memory Update)
```

### Parallel Execution
When multiple tasks become unblocked simultaneously, assign to multiple teammates at once.

---

## Memory Update (Workflow-Final)

**ALWAYS execute after team completes:**

1. Collect `### Memory Notes` sections from ALL teammate outputs
2. Persist learnings to `.claude/cc100x/activeContext.md ## Learnings`
3. Persist patterns to `.claude/cc100x/patterns.md ## Common Gotchas`
4. Persist verification to `.claude/cc100x/progress.md ## Verification`

**Use Read-Edit-Read pattern for each file.**

---

## Gates (Must Pass)

1. **MEMORY_LOADED** - Before routing
2. **TASKS_CHECKED** - Check TaskList() for active workflow
3. **INTENT_CLARIFIED** - User intent is unambiguous
4. **TEAM_CREATED** - Agent Team spawned for workflow
5. **ALL_TASKS_COMPLETED** - All tasks (including Memory Update) completed
6. **CONTRACTS_VALIDATED** - All Router Contracts parsed and validated
7. **MEMORY_UPDATED** - Before marking done

---

## Team Shutdown

After workflow completes:
1. Send shutdown_request to all teammates
2. Wait for shutdown approvals
3. Clean up team: `TeamDelete()`
4. Report results to user

---

## Key Rules

1. **Lead NEVER implements code** - delegate mode always
2. **Every teammate outputs a Router Contract** - YAML at end of output
3. **Every workflow reads memory first** - `.claude/cc100x/`
4. **Teammates own file sets** - no two teammates edit the same file
5. **READ-ONLY agents** include Memory Notes for lead to persist
6. **WRITE agents** update memory directly
7. **Wait for teammates** - never implement while teammates are working
8. **All skill references** use `cc100x:` prefix
