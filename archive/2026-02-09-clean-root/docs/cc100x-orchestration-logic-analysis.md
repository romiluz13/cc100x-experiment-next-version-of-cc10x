# CC100x Orchestration Logic Analysis

> **Last synced with agents/skills:** 2026-02-06 | **Status:** IN SYNC

> **Relationship to Bible:** This document explains HOW the system works.
> For the canonical specification (WHAT the system IS), see `docs/cc100x-bible.md`.

> **Critical Understanding:** This is NOT code orchestration. This is **English orchestration** — prompt engineering and instructions that guide AI behavior. Every "chain," "gate," and "handoff" is implemented through carefully crafted English text. CC100x adds **Agent Teams** as a structural coordination layer (teams, teammates, messaging, shared tasks) on top of English orchestration.

---

## System Architecture Overview

### Components

```
┌─────────────────────────────────────────────────────────────────────┐
│                      cc100x-lead (TEAM LEAD)                        │
│   THE ONLY ENTRY POINT - Detects intent, routes to workflows        │
│   Decision Tree: ERROR -> DEBUG | PLAN -> PLAN | REVIEW -> REVIEW | -> BUILD │
│   Creates teams, spawns teammates, validates Router Contracts       │
└─────────────────────────────────────────────────────────────────────┘
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         ▼                          ▼                          ▼
    ┌─────────┐                ┌─────────┐                ┌─────────┐
    │ AGENTS  │                │ SKILLS  │                │ MEMORY  │
    │ (9)     │                │ (16)    │                │ (3 files)│
    └─────────┘                └─────────┘                └─────────┘
```

### The 9 Agents (Executors)
| Agent | Role | Workflow Usage | Mode | Memory | Model |
|-------|------|----------------|------|--------|-------|
| `builder` | Build features using TDD | BUILD | WRITE | Direct Edit | inherit |
| `security-reviewer` | Security-focused code review | REVIEW, BUILD post-review | READ-ONLY | Memory Notes | inherit |
| `performance-reviewer` | Performance-focused code review | REVIEW, BUILD post-review | READ-ONLY | Memory Notes | inherit |
| `quality-reviewer` | Quality/correctness code review | REVIEW, BUILD post-review, DEBUG post-fix | READ-ONLY | Memory Notes | inherit |
| `live-reviewer` | Real-time review concurrent with builder | BUILD (concurrent with builder) | READ-ONLY | Memory Notes | inherit |
| `hunter` | Find error handling gaps | BUILD, DEBUG | READ-ONLY | Memory Notes | inherit |
| `verifier` | E2E validation | BUILD, DEBUG | READ-ONLY | Memory Notes | inherit |
| `investigator` | Root cause debugging | DEBUG | READ-ONLY | Memory Notes | inherit |
| `planner` | Create implementation plans | PLAN | WRITE | Direct Edit | inherit |

**Mode Clarification:**
- **WRITE agents** (builder, planner): Have Edit tool, update `.claude/cc100x/*.md` memory directly, load session-memory skill via frontmatter
- **READ-ONLY agents** (security-reviewer, performance-reviewer, quality-reviewer, live-reviewer, hunter, verifier, investigator): No Edit tool, output `### Memory Notes` section, persisted via task-enforced Memory Update task
- **Memory Notes pattern**: READ-ONLY agents include learnings/patterns/verification in structured output section

### The 16 Skills (Knowledge/Patterns)
| Skill | Purpose | Loaded Via |
|-------|---------|------------|
| `cc100x-lead` | THE ENTRY POINT - orchestration engine | Main assistant |
| `session-memory` | Memory persistence rules | Frontmatter (WRITE agents: builder, planner) |
| `router-contract` | Router Contract output format + validation | Frontmatter (ALL agents) |
| `verification` | Evidence requirements | Frontmatter (ALL agents) |
| `review-arena` | Parallel adversarial code review protocol | SKILL_HINTS (reviewers) |
| `bug-court` | Competing hypothesis debugging protocol | SKILL_HINTS (investigator, hunter, verifier) |
| `pair-build` | Real-time pair programming protocol | SKILL_HINTS (builder, live-reviewer) |
| `planning-patterns` | How to write plans | SKILL_HINTS (planner) |
| `debugging-patterns` | Systematic debugging process | SKILL_HINTS (investigator, hunter, verifier) |
| `code-review-patterns` | Review methodology | SKILL_HINTS (reviewers, live-reviewer, hunter) |
| `code-generation` | Code writing guidelines | SKILL_HINTS (builder) |
| `test-driven-development` | TDD cycle | SKILL_HINTS (builder, investigator) |
| `architecture-patterns` | System design | SKILL_HINTS (per-workflow) |
| `brainstorming` | Ideas to designs | SKILL_HINTS (planner) |
| `frontend-patterns` | UI/UX patterns | SKILL_HINTS (per-workflow) |
| `github-research` | External research (conditional) | SKILL_HINTS (planner, investigator) |

### The 3 Memory Files
| File | Purpose | When Updated |
|------|---------|--------------|
| `.claude/cc100x/activeContext.md` | Current focus, decisions, learnings | Every workflow |
| `.claude/cc100x/patterns.md` | Project conventions, gotchas | When patterns learned |
| `.claude/cc100x/progress.md` | What's done, what's remaining | Task completion |

---

## Orchestration Flow (Detailed)

### Phase 0: Lead Activation

**Trigger Keywords (from SKILL.md description):**
```
build, implement, create, make, write, add, develop, code, feature,
component, app, application, review, audit, check, analyze, debug,
fix, error, bug, broken, troubleshoot, plan, design, architect,
roadmap, strategy, memory, session, context, save, load, test, tdd,
frontend, ui, backend, api, pattern, refactor, optimize, improve,
enhance, update, modify, change, help, assist, work, start, begin,
continue, research, cc100x, c100x
```

### Phase 1: Memory Loading (GATE: MEMORY_LOADED)

```
Bash(command="mkdir -p .claude/cc100x")
Read(file_path=".claude/cc100x/activeContext.md")
Read(file_path=".claude/cc100x/patterns.md")
Read(file_path=".claude/cc100x/progress.md")
```

**English Trick:** Separating mkdir and Read prevents permission prompts (compound commands ask for permission, separate tools don't).

### Phase 2: Task State Check (GATE: TASKS_CHECKED)

```
TaskList()  # Check for pending/in-progress workflow tasks
```

**Decision Point:**
- If active CC100x workflow task exists -> Resume from task state
- If no active tasks -> Proceed with workflow selection

### Phase 3: Intent Classification

**Decision Tree (PRIORITY ORDER - ERROR always wins):**

| Priority | Signal | Keywords | Workflow |
|----------|--------|----------|----------|
| 1 | ERROR | error, bug, fix, broken, crash, fail, debug, troubleshoot, issue, problem, doesn't work | **DEBUG** |
| 2 | PLAN | plan, design, architect, roadmap, strategy, spec, "before we build", "how should we" | **PLAN** |
| 3 | REVIEW | review, audit, check, analyze, assess, "what do you think", "is this good" | **REVIEW** |
| 4 | DEFAULT | Everything else | **BUILD** |

**English Trick:** "Conflict Resolution: ERROR signals always win" prevents ambiguity. "fix the build" = DEBUG (not BUILD).

### Phase 4: Skill Loading

CC100x uses a **two-mechanism approach** for skill loading, unlike CC10x which loaded all skills via frontmatter:

**Mechanism 1: Frontmatter (Automatic, Protocol Skills)**

These load automatically when the agent starts - they define HOW agents communicate and persist state:

| Skill | Loaded By | Purpose |
|-------|-----------|---------|
| `router-contract` | ALL agents | Ensures every agent outputs a valid Router Contract |
| `verification` | ALL agents | Evidence requirements for completion claims |
| `session-memory` | WRITE agents only (builder, planner) | Memory persistence rules |

**Mechanism 2: SKILL_HINTS (Per-Workflow, Domain Skills)**

These are passed by the lead in the agent prompt via `SKILL_HINTS:` - they provide domain expertise:

| Skill | Triggered When | Agents |
|-------|----------------|--------|
| `cc100x:test-driven-development` | BUILD, DEBUG | builder, investigator |
| `cc100x:code-generation` | BUILD | builder |
| `cc100x:code-review-patterns` | REVIEW, BUILD post-review, DEBUG post-fix | security-reviewer, performance-reviewer, quality-reviewer, live-reviewer, hunter |
| `cc100x:debugging-patterns` | DEBUG | investigator, hunter, verifier |
| `cc100x:planning-patterns` | PLAN | planner |
| `cc100x:brainstorming` | PLAN | planner |
| `cc100x:architecture-patterns` | All workflows (as needed) | Per-workflow agents |
| `cc100x:frontend-patterns` | All workflows (as needed) | Per-workflow agents |
| `cc100x:review-arena` | REVIEW | security-reviewer, performance-reviewer, quality-reviewer |
| `cc100x:bug-court` | DEBUG | investigator, hunter, verifier |
| `cc100x:pair-build` | BUILD | builder, live-reviewer |
| `cc100x:github-research` | External/exhausted/explicit | planner, investigator |

**Conditional Trigger for github-research (unchanged from SKILL_HINTS):**

| Trigger | Skill | Agents |
|---------|-------|--------|
| External: new tech (post-2024), unfamiliar library, complex integration | `cc100x:github-research` | planner, investigator |
| Debug exhausted: 3+ local attempts failed | `cc100x:github-research` | investigator |
| User explicit: "research", "github", "octocode" | `cc100x:github-research` | planner, investigator |

**Flow:** Lead detects trigger -> passes `github-research` in SKILL_HINTS -> Agent calls `Skill(skill="cc100x:github-research")`

### Phase 5: Workflow-Specific Execution

#### BUILD Workflow

```
Chain: builder + live-reviewer (concurrent) -> hunter -> verifier
                    ↑ PAIR BUILD ↑
```

**Steps:**
1. Load memory -> Check progress.md (already done?)
2. **Clarify requirements** -> AskUserQuestion (GATE: REQUIREMENTS_CLARIFIED)
3. **Create task hierarchy** (GATE: TASKS_CREATED)
4. **Chain execution loop** (builder and live-reviewer run concurrently via Pair Build)
5. Update memory when ALL tasks completed (GATE: MEMORY_UPDATED)

#### DEBUG Workflow

```
Chain: investigators (parallel, 2-5) -> debate -> builder fix -> quality-reviewer -> hunter -> verifier
                ↑ BUG COURT ↑
```

**Steps:**
1. Load memory -> Check patterns.md Common Gotchas
2. **Clarify** (REQUIRED): What error? Expected vs actual? When started?
3. **Check research triggers:**
   - User explicitly requested research? OR
   - External service error? OR
   - 3+ local debugging attempts failed?
   - **If ANY -> Execute research FIRST, persist to docs/research/, update memory**
4. Create task hierarchy (Bug Court: multiple investigators with competing hypotheses)
5. Chain execution (debate phase, then builder fix, then review + verification)
6. Update memory -> Add to Common Gotchas

**Debug Attempt Tracking Format:**
```
[DEBUG-N]: {what was tried} -> {result}
```
Example entries in activeContext.md Recent Changes:
- `[DEBUG-1]: Added null check -> still failing (same error)`
- `[DEBUG-2]: Wrapped in try-catch -> error is in upstream fetch()`
- `[DEBUG-3]: Fixed fetch() URL encoding -> tests pass`

Lead counts `[DEBUG-N]:` lines to trigger external research after 3+ failures.

#### REVIEW Workflow

```
Chain: security-reviewer || performance-reviewer || quality-reviewer -> challenge round -> consensus
                              ↑ REVIEW ARENA ↑
```

**Steps:**
1. Load memory
2. **Clarify** (REQUIRED): Entire codebase OR specific files? Focus area?
3. Create task hierarchy (Review Arena: three parallel reviewers)
4. Execute (parallel review -> challenge round via peer messaging -> consensus)
5. Update memory

#### PLAN Workflow

```
Chain: planner (single agent)
```

**Steps:**
1. Load memory
2. **If github-research detected:**
   - Execute research FIRST using octocode tools (NOT as hint)
   - **PERSIST research** -> docs/research/YYYY-MM-DD-<topic>-research.md
   - **Update memory** -> activeContext.md Research References table
   - Summarize findings before invoking planner
3. Create task hierarchy
4. Execute (pass research results + file path in prompt)
5. Update memory -> Reference saved plan

---

## Chain Execution Loop (THE HEART OF ORCHESTRATION)

**English Pattern for Sequential + Parallel Execution with Agent Teams:**

```
SETUP:
1. Lead creates team:
   TeamCreate(team_name="{workflow}-{timestamp}")

LOOP:
1. Find runnable tasks:
   TaskList() -> Find tasks where:
   - status = "pending"
   - blockedBy is empty OR all blockedBy tasks are "completed"

2. Spawn teammate(s):
   - TaskUpdate(taskId, status="in_progress")
   - If MULTIPLE tasks ready (e.g., security-reviewer + performance-reviewer + quality-reviewer):
     -> Spawn ALL in SAME MESSAGE (parallel execution)
   - Pass task ID in prompt:
     Task(subagent_type="cc100x:{agent}", team_name="{team}", name="{agent-name}", prompt="
       Your task ID: {taskId}
       User request: {request}
       Requirements: {requirements}
       Memory: {activeContext}
       SKILL_HINTS: {detected skills}
     ")

3. After teammate completes:
   - Lead calls TaskUpdate(taskId, status="completed")
   - Lead validates Router Contract from teammate output
   - Lead calls TaskList() to find next tasks

4. Determine next:
   - Find tasks where ALL blockedBy tasks are "completed"
   - If multiple ready -> Spawn ALL in parallel (same message)
   - If one ready -> Spawn sequentially
   - If none ready AND uncompleted tasks exist -> Error state
   - If ALL tasks completed -> Workflow complete

5. Repeat until:
   - All tasks have status="completed"
   - OR critical error detected

TEARDOWN:
   - Lead sends shutdown_request to all teammates
   - TeamDelete() to clean up team resources
```

**Critical English Trick:** "Both Task calls in same message = both complete before you continue" - This leverages Claude's tool execution model where parallel tool calls in one message complete before the next response.

**Agent Teams Addition:** The lead operates in **delegate mode** - it NEVER implements code directly. It creates teams, spawns teammates, validates their Router Contracts, and coordinates handoffs. Teammates communicate via SendMessage for peer interactions (Review Arena challenge rounds, Bug Court debates, Pair Build feedback).

---

## Task Hierarchy Structures

### BUILD Workflow Tasks

```
[Parent]      BUILD: {feature_summary}
                   |
         ┌────────┴────────┐
         ▼                  ▼
[Agent 1a]              [Agent 1b]
builder              live-reviewer
(concurrent)         (concurrent, READ-ONLY)
         |                  |
         └────────┬─────────┘
                  ▼
[Agent 2]     hunter
              (blockedBy: 1a, 1b)
                  |
                  ▼
[Agent 3]     verifier
              (blockedBy: 2)
                  |
                  ▼
[Task]        Memory Update           <- TASK-ENFORCED
              (blockedBy: 3)
```

### DEBUG Workflow Tasks

```
[Parent]      DEBUG: {error_summary}
                   |
    ┌──────────────┼──────────────┐
    ▼              ▼              ▼
[Agent 1a]    [Agent 1b]    [Agent 1c]
investigator  investigator  investigator
(parallel, competing hypotheses - Bug Court)
    |              |              |
    └──────────────┼──────────────┘
                   ▼
[Phase]       Debate (peer messaging via SendMessage)
                   |
                   ▼
[Agent 2]     builder: Apply fix (blockedBy: debate)
                   |
                   ▼
[Agent 3]     quality-reviewer: Review fix (blockedBy: 2)
                   |
                   ▼
[Agent 4]     hunter: Check for gaps (blockedBy: 3)
                   |
                   ▼
[Agent 5]     verifier: Verify fix (blockedBy: 4)
                   |
                   ▼
[Task]        Memory Update (blockedBy: 5)  <- TASK-ENFORCED
```

### REVIEW Workflow Tasks

```
[Parent]      REVIEW: {scope_summary}
                   |
    ┌──────────────┼──────────────┐
    ▼              ▼              ▼
[Agent 1a]    [Agent 1b]    [Agent 1c]
security-     performance-  quality-
reviewer      reviewer      reviewer
(parallel - Review Arena)
    |              |              |
    └──────────────┼──────────────┘
                   ▼
[Phase]       Challenge Round (peer messaging via SendMessage)
                   |
                   ▼
[Phase]       Consensus
                   |
                   ▼
[Task]        Memory Update  <- TASK-ENFORCED
```

---

## English Tricks & Patterns Used

### 1. Permission-Free Operations
Using specific tools to avoid permission prompts:
- `mkdir -p` as single command (no compound)
- `Read()` tool instead of `cat` command
- `Edit()` for updates instead of `Write()` for overwrites

### 2. Gate Enforcement Through English
"GATE: X" creates psychological checkpoints:
- MEMORY_LOADED - Before routing
- TASKS_CHECKED - Check TaskList() for active workflow
- INTENT_CLARIFIED - User intent is unambiguous
- RESEARCH_EXECUTED - Before planner (if github-research detected)
- RESEARCH_PERSISTED - Save + update memory
- REQUIREMENTS_CLARIFIED - Before invoking agent (BUILD only)
- TASKS_CREATED - Workflow task hierarchy created
- ALL_TASKS_COMPLETED - All agent tasks completed
- MEMORY_UPDATED - Before marking done

### 3. Confidence Scoring
"Only report issues with confidence >=80" - Prevents vague feedback.

### 4. Rationalization Prevention Tables
Every skill has a table mapping excuses to reality:
```
| "I'll test later" | Tests passing immediately prove nothing |
```

### 5. Iron Laws
Each skill has a single, memorable rule that cannot be violated:
- session-memory: "LOAD memory at START, UPDATE at END"
- debugging-patterns: "NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST"
- code-review-patterns: "NO CODE QUALITY REVIEW BEFORE SPEC COMPLIANCE"
- test-driven-development: "NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST"
- code-generation: "NO CODE BEFORE UNDERSTANDING FUNCTIONALITY AND PROJECT PATTERNS"
- planning-patterns: "NO VAGUE STEPS - EVERY STEP IS A SPECIFIC ACTION"
- architecture-patterns: "NO ARCHITECTURE DESIGN BEFORE FUNCTIONALITY FLOWS ARE MAPPED"
- frontend-patterns: "NO UI DESIGN BEFORE USER FLOW IS UNDERSTOOD"
- brainstorming: "NO DESIGN WITHOUT UNDERSTANDING PURPOSE AND CONSTRAINTS"
- verification: "NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE"
- github-research: "NO EXTERNAL RESEARCH WITHOUT CLEAR AI KNOWLEDGE GAP OR EXPLICIT USER REQUEST"
- review-arena: "NO CONSENSUS WITHOUT INDEPENDENT REVIEW FIRST"
- bug-court: "NO FIX WITHOUT COMPETING HYPOTHESES EVALUATED"
- pair-build: "NO CODE MERGED WITHOUT LIVE REVIEW FEEDBACK ADDRESSED"
- router-contract: "NO AGENT OUTPUT WITHOUT MACHINE-READABLE CONTRACT"

### 6. Agent-Specific Gates (Beyond Lead Gates)

**builder: Plan File Check**
- If `Plan File` in prompt is NOT "None" -> MUST read plan file first
- Match task to plan's phases/steps
- Follow plan's specific instructions
- Cannot proceed without reading plan

**investigator: Anti-Hardcode Gate**
- Before writing regression test, identify variant dimensions:
  - Locale/i18n, config/env, roles/permissions, platform, time, data shape, concurrency, network, caching
- Regression test MUST cover at least one non-default variant case
- Prevents patchy/hardcoded fixes

### 7. Output Formats as Templates
Every agent/skill has a specific output format with sections:
- Forces structured thinking
- Ensures nothing is forgotten
- Makes handoffs consistent

---

## Agent -> Skill Mapping

### Which Skills Each Agent Uses

CC100x uses a two-mechanism approach: **protocol skills** load via frontmatter (automatic), **domain skills** load via SKILL_HINTS (per-workflow). Only `github-research` has additional conditional triggers.

| Agent | Mode | Protocol Skills (Frontmatter) | Domain Skills (SKILL_HINTS) | Conditional |
|-------|------|-------------------------------|----------------------------|-------------|
| builder | WRITE | router-contract, verification, session-memory | test-driven-development, code-generation, pair-build, architecture-patterns, frontend-patterns | -- |
| security-reviewer | READ-ONLY | router-contract, verification | code-review-patterns, review-arena, architecture-patterns | -- |
| performance-reviewer | READ-ONLY | router-contract, verification | code-review-patterns, review-arena, architecture-patterns | -- |
| quality-reviewer | READ-ONLY | router-contract, verification | code-review-patterns, review-arena, architecture-patterns, frontend-patterns | -- |
| live-reviewer | READ-ONLY | router-contract, verification | code-review-patterns, pair-build, code-generation, architecture-patterns, frontend-patterns | -- |
| hunter | READ-ONLY | router-contract, verification | code-review-patterns, debugging-patterns, architecture-patterns, frontend-patterns | -- |
| verifier | READ-ONLY | router-contract, verification | debugging-patterns, architecture-patterns, frontend-patterns | -- |
| investigator | READ-ONLY | router-contract, verification | debugging-patterns, bug-court, test-driven-development, architecture-patterns, frontend-patterns | github-research (external/exhausted) |
| planner | WRITE | router-contract, verification, session-memory | planning-patterns, architecture-patterns, brainstorming, frontend-patterns | github-research (external tech) |

**Notes:**
- READ-ONLY agents don't load session-memory. They output `### Memory Notes` section; persisted via task-enforced "CC100X Memory Update" task at workflow-final.
- Only `github-research` uses conditional `Skill()` calls - protocol skills load automatically via frontmatter, domain skills load via SKILL_HINTS from the lead.
- **Workflow-specific skills** (review-arena, bug-court, pair-build) are only passed as SKILL_HINTS when the corresponding workflow is active.

### Context Retrieval Pattern (Used by planner, investigator)

**When exploring unfamiliar or large codebases:**

```
Cycle 1: DISPATCH - Broad search (grep patterns, related keywords)
Cycle 2: EVALUATE - Score relevance (0-1), note codebase terminology
Cycle 3: REFINE - Focus on high-relevance (>=0.7), fill gaps
Max 3 cycles, then proceed with best available context
```

**Stop when:** Understand existing patterns, dependencies, and constraints (planner) OR 3+ files with relevance >=0.7 AND no critical gaps (investigator)

**English Trick:** Bounded exploration prevents infinite context gathering while ensuring adequate understanding.

---

## Memory Flow

### Load -> Work -> Update Cycle

```
SESSION START
      │
      ▼
┌─────────────────┐
│ LOAD MEMORY     │ Read all 3 files
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ CHECK MEMORY    │ Before each decision
│ BEFORE DECISION │ - Did we decide this before?
└────────┬────────┘ - Is there a project pattern?
         │
         ▼
┌─────────────────┐
│ DO WORK         │ Execute workflow
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ UPDATE MEMORY   │ Edit (not Write!) for permission-free
└────────┬────────┘
         │
         ▼
   SESSION END
```

### Memory File Purposes

**activeContext.md** - What's happening NOW
- `## Current Focus` - Active work
- `## Recent Changes` - What changed (includes `[DEBUG-N]:` format for debug tracking)
- `## Next Steps` - What's next
- `## Decisions` - Choices made and why
- `## Learnings` - Insights discovered
- `## References` - Links to Plan, Design, Research files
- `## Blockers` - What's blocking progress
- `## Last Updated` - Timestamp

**patterns.md** - What we've LEARNED
- `## Architecture Patterns` - How this project implements patterns
- `## Code Conventions` - Naming, style, structure
- `## File Structure` - Where things go
- `## Testing Patterns` - How to write tests here
- `## Common Gotchas` - Bugs and solutions (including from research)
- `## API Patterns` - Endpoint conventions
- `## Error Handling` - How project handles errors
- `## Dependencies` - What's used and why

**progress.md** - What's DONE
- `## Current Workflow` - PLAN | BUILD | REVIEW | DEBUG
- `## Tasks` - Active task list with checkboxes
- `## Completed` - Done items with evidence
- `## Verification` - Commands + exit codes
- `## Last Updated` - Timestamp

---

## Research Flow (External)

### Three-Phase Pattern

```
Phase 1: Execute Research
         │
         ▼ (ATOMIC - no agent invocations between phases)
Phase 2: Persist Research
         - Bash("mkdir -p docs/research")
         - Write("docs/research/YYYY-MM-DD-<topic>-research.md")
         - Edit(".claude/cc100x/activeContext.md") -> Add to Research References
         │
         ▼
Phase 3: Invoke Agent with Research Context
         - Task(cc100x:planner, prompt="...Research findings: {results}...")
```

**Critical:** "Research without persistence is LOST after context compaction."

---

## Handoff Patterns

### Lead -> Teammate Handoff

```
Task(subagent_type="cc100x:builder", team_name="{team}", name="builder", prompt="
Your task ID: {taskId}
User request: {request}
Requirements: {from AskUserQuestion}
Memory: {from activeContext.md}
Patterns: {from patterns.md}
SKILL_HINTS: {detected skills from table}
")
```

**Task ID is REQUIRED in prompt.** Lead updates task status after teammate returns (teammates do NOT call TaskUpdate for own task).
**SKILL_HINTS are MANDATORY** - Teammate must load each skill immediately.

### Teammate -> Lead Handoff (Completion)

**All teammates output two mandatory sections:**

1. **Dev Journal (User Transparency)** - Human-readable narrative:
   - What I Did (actions taken)
   - Key Decisions Made (decisions + WHY)
   - Alternatives Considered (what was rejected + reason)
   - Where Your Input Helps (decision points, assumptions to validate)
   - What's Next (what user should expect from next phase)

2. **Router Contract (Machine-Readable)** - YAML block for validation:
   ```yaml
   STATUS: [PASS|FAIL|APPROVE|CHANGES_REQUESTED|CLEAN|ISSUES_FOUND|FIXED|PLAN_CREATED]
   CONFIDENCE: [0-100]
   CRITICAL_ISSUES: [count]
   BLOCKING: [true|false]
   REQUIRES_REMEDIATION: [true|false]
   REMEDIATION_REASON: [null or exact text for REM-FIX task]
   MEMORY_NOTES:
     learnings: ["..."]
     patterns: ["..."]
     verification: ["..."]
   ```

**Router Contract enables:**
- Machine-readable validation (no fragile string parsing)
- Consistent remediation handling across all teammates
- Memory Notes collection for workflow-final persistence

**WRITE teammates** (builder, planner):
1. Complete the work
2. Update memory directly using `Edit()` + `Read()` verify
3. Return structured output with Dev Journal + Router Contract

**READ-ONLY teammates** (security-reviewer, performance-reviewer, quality-reviewer, live-reviewer, hunter, verifier, investigator):
1. Complete the analysis/verification
2. Include Memory Notes in Router Contract YAML
3. Return structured output with Dev Journal + Router Contract

**Lead's responsibility after Task() returns:**
1. Call `TaskUpdate(taskId, status="completed")` (teammates do NOT do this)
2. **Validate Router Contract:**
   - Look for `### Router Contract (MACHINE-READABLE)` section
   - Parse YAML block
   - If `BLOCKING=true` or `REQUIRES_REMEDIATION=true` -> Create REM-FIX task, block downstream
   - Circuit breaker: If 3+ REM-FIX tasks exist -> AskUserQuestion for direction
3. Collect `MEMORY_NOTES` from contract for workflow-final persistence
4. Call `TaskList()` to find next tasks
5. Find next runnable tasks (including "CC100X Memory Update" task)
6. Continue loop or complete workflow
7. **When Memory Update task becomes available:** Persist collected Memory Notes to `.claude/cc100x/*.md` (task-enforced, survives context compaction)

### Peer Messaging Pattern (Agent Teams Addition)

Teammates communicate directly with each other via `SendMessage` for structured interactions:

**Review Arena (REVIEW workflow):**
- Lead spawns security-reviewer, performance-reviewer, quality-reviewer in parallel
- Each reviewer completes independent analysis
- Lead triggers challenge round: reviewers exchange findings via `SendMessage`
- Each reviewer can challenge or endorse other reviewers' findings
- Lead collects final consensus

**Bug Court (DEBUG workflow):**
- Lead spawns multiple investigators in parallel with competing hypotheses
- Investigators complete independent root cause analysis
- Lead triggers debate: investigators exchange hypotheses via `SendMessage`
- Investigators challenge each other's root cause theories
- Lead selects winning hypothesis, spawns builder for fix

**Pair Build (BUILD workflow):**
- Lead spawns builder and live-reviewer concurrently
- live-reviewer monitors builder's changes in real-time
- live-reviewer sends feedback to builder via `SendMessage`
- Builder addresses feedback inline during implementation
- Both complete before downstream tasks (hunter, verifier) begin

### Plan -> Build Handoff

When plan is created:
1. Plan saved to `docs/plans/YYYY-MM-DD-<feature>-plan.md`
2. Memory updated with reference
3. User asked: "Execute now?" or "Manual execution?"

When executing plan:
1. builder receives `planFile` in task metadata
2. builder reads plan file
3. Follows plan task-by-task

---

## Verification Flow

### Before Any Completion Claim

```
IDENTIFY: What command proves this claim?
    │
    ▼
RUN: Execute the FULL command (fresh, complete)
    │
    ▼
READ: Full output, check exit code, count failures
    │
    ▼
VERIFY: Does output confirm the claim?
    │
    ├──▶ NO: State actual status with evidence
    │
    └──▶ YES: State claim WITH evidence
```

### Goal-Backward Lens (After Standard Verification)

```
GOAL: [What user wants to achieve]

TRUTHS (observable):
- [ ] [User-facing behavior 1]
- [ ] [User-facing behavior 2]

ARTIFACTS (exist):
- [ ] [Required file/endpoint 1]
- [ ] [Required file/endpoint 2]

WIRING (connected):
- [ ] [Component] -> [calls] -> [API]
- [ ] [API] -> [queries] -> [Database]

Standard verification: exit code 0
Goal check: All boxes checked?
```

---

## Task Coordination Mechanics

### The Hydration Pattern (Critical)

CC100x uses BOTH Tasks AND Memory files for different purposes:

```
┌─────────────────────┐     Session Start      ┌──────────────────┐
│  Memory Files       │ ────────────────────►  │  Claude Tasks    │
│  (progress.md)      │      "Hydrate"         │  (session state) │
└─────────────────────┘                        └──────────────────┘
                                                       │
                                                       │ Work
                                                       ▼
┌─────────────────────┐     Session End        ┌──────────────────┐
│  Memory Files       │  ◄──────────────────── │  Task Updates    │
│  (updated)          │      "Sync back"       │  (completed)     │
└─────────────────────┘                        └──────────────────┘
```

**Why this matters:**
- Tasks may be session-scoped (not guaranteed to persist across sessions)
- Memory files ARE persistent (survive sessions, stored in `.claude/cc100x/`)
- CC100x uses BOTH: Tasks for runtime coordination, memory for persistence
- At session start: Create tasks from progress.md state
- At session end: Sync task status back to progress.md

**The key insight:** Tasks are the execution engine; memory is the persistence layer.

**Agent Teams Addition:** Agent Teams tasks are shared within the team via `~/.claude/tasks/{team-name}/`. All teammates in a team can read and update the shared task list, enabling real-time coordination. When the team is deleted (TeamDelete), these task files are cleaned up automatically. Memory files in `.claude/cc100x/` persist independently of teams.

### Cross-Session Coordination

Agent Teams provides structured cross-session coordination through teams:

```
Team creation: TeamCreate(team_name="{workflow}-{timestamp}")
Shared tasks:  ~/.claude/tasks/{team-name}/
Team config:   ~/.claude/teams/{team-name}/config.json
```

**What this enables:**
- Teammate A (builder) completes Task #1
- Teammate B (live-reviewer) sees Task #2 is now unblocked
- All teammates share state in real-time within the team

**CC100x Safety Stance:**
- Agent Teams tasks are team-scoped and shared among teammates
- CC100x treats Tasks as **team-scoped and session-lived**
- Therefore: namespace with `CC100X ` prefix, keep examples schema-minimal
- Always scope before resuming (check if tasks belong to current project)
- Teams are created per-workflow and cleaned up after completion

### When to Use Tasks (Decision Guide)

**USE Tasks when:**
| Scenario | Why |
|----------|-----|
| Multi-file features | Track progress across files |
| Large-scale refactors | Ensure nothing missed |
| Parallelizable work | Enable concurrent teammates |
| Complex dependencies | Automatic unblocking |
| Teammate coordination | Shared progress tracking |

**SKIP Tasks when:**
| Scenario | Why |
|----------|-----|
| Single-function fixes | Overhead exceeds benefit |
| Simple bugs | Just fix it directly |
| Trivial edits | No tracking needed |
| < 3 steps | Not worth orchestration |

**The 3-Task Rule:** If you have fewer than 3 related steps, just do them directly.

---

## Summary: The Orchestration is Pure English + Agent Teams

Every aspect of this system is English-based, with Agent Teams adding structured coordination:

1. **Intent Detection** = Keyword matching in English
2. **Workflow Selection** = Decision tree described in English
3. **Gate Enforcement** = "GATE:" labels with psychological weight
4. **Chain Execution** = Instructions about task dependencies
5. **Parallel Execution** = "Both Task calls in same message"
6. **Memory Persistence** = Specific tool usage instructions
7. **Agent Behavior** = Templates and rationalization prevention
8. **Verification** = "NO completion claims without evidence"
9. **Team Coordination** = Agent Teams (teams, teammates, messaging, shared tasks)
10. **Peer Review** = SendMessage between teammates (Review Arena, Bug Court, Pair Build)
11. **Delegate Mode** = Lead never implements, only orchestrates via teammates

The "code" is the English. The "compiler" is Claude's instruction following. The "bugs" are ambiguities, gaps, and conflicts in the English instructions. Agent Teams adds a **structural coordination layer** -- teams, messaging, shared task lists -- that makes the English orchestration concrete and executable across multiple concurrent agents.
