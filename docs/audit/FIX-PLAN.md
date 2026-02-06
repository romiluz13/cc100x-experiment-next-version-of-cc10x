# CC10x → CC100x Fix Plan (Post-Audit)

> **Context:** This plan fixes all findings from the CC10x→CC100x audit (see `findings.md`).
> **Foundation:** Agent Teams is the core engine. All fixes use Agent Teams conventions.
> **Principle:** CC100x distributes skills via SKILL_HINTS (lead → teammates per workflow),
>   not frontmatter (unconditional preload). This is intentional for Agent Teams token economy.
> **Source files:** All CC10x originals are in `reference/cc10x-skills/`.

---

## Phase A: Create 3 Missing Skills [P0 — CRITICAL]

Three CC10x skills (1,309 lines combined) were dropped entirely. Port each from CC10x with mechanical name changes.

### Mechanical Name Changes (Apply to ALL 3 skills)

These are the EXACT text replacements to apply across all ported skills:

| Find | Replace | Context |
|------|---------|---------|
| `"Internal skill. Use cc10x-router for all development tasks."` | `"Internal skill. Use cc100x-lead for all development tasks."` | Frontmatter description |
| `.claude/cc10x/` | `.claude/cc100x/` | Memory file paths |
| `cc10x-router` | `cc100x-lead` | Orchestrator reference in prose |
| `CC10X ` | `CC100X ` | Task prefixes (note trailing space) |
| `cc10x:` | `cc100x:` | Skill references (e.g., `Skill(skill="cc10x:brainstorming")`) |
| `component-builder` | `builder` | Agent name in prose |
| `bug-investigator` | `investigator` | Agent name in prose |
| `code-reviewer` | `quality-reviewer` | Agent name in prose (context-dependent) |
| `silent-failure-hunter` | `hunter` | Agent name in prose |
| `integration-verifier` | `verifier` | Agent name in prose |
| `subagent` / `sub-agent` | `teammate` | Agent Teams terminology |
| `router` (when meaning orchestrator) | `lead` | Agent Teams terminology |

### A1: Create `architecture-patterns` skill

**Source:** `reference/cc10x-skills/architecture-patterns.md` (361 lines)
**Target:** `plugins/cc100x/skills/architecture-patterns/SKILL.md`
**Action:** Copy source to target, apply ALL mechanical name changes from table above.

**Verify after creation:**
- Line 3: `description: "Internal skill. Use cc100x-lead for all development tasks."`
- No remaining occurrences of `cc10x` anywhere in the file
- All Iron Law, Universal Questions, Phase 1-3, Architecture Views, LSP section, API Design, Integration Patterns, Observability, Decision Framework, Rationalization Prevention, Output Format, Final Check — ALL preserved

### A2: Create `frontend-patterns` skill

**Source:** `reference/cc10x-skills/frontend-patterns.md` (583 lines)
**Target:** `plugins/cc100x/skills/frontend-patterns/SKILL.md`
**Action:** Copy source to target, apply ALL mechanical name changes from table above.

**Verify after creation:**
- Line 3: `description: "Internal skill. Use cc100x-lead for all development tasks."`
- No remaining occurrences of `cc10x` anywhere in the file
- ALL of these sections preserved (check each):
  - Iron Law ("NO UI DESIGN BEFORE USER FLOW IS UNDERSTOOD")
  - Design Thinking (Purpose, Tone, Constraints, Differentiation)
  - Loading State Order (Error→Loading→Empty→Data)
  - Skeleton vs Spinner
  - Motion & Animation (prefers-reduced-motion)
  - Error Handling Hierarchy (inline→toast→banner→full screen)
  - Typography Rules (ellipsis, quotes, units, tabular-nums)
  - Content Overflow Handling (truncate, line-clamp, min-w-0)
  - Universal Questions
  - User Flow First
  - UX Review Checklist
  - Accessibility Review (WCAG 2.1 AA)
  - Form Best Practices (autocomplete, never block paste)
  - Visual Design + Creativity (anti-AI-slop)
  - Spatial Composition (asymmetry, overlap, grid-breaking)
  - Component Patterns (Buttons, Forms, Loading, Errors code examples)
  - Responsive Design Checklist
  - Performance Rules (virtualize, lazy load, preconnect, preload)
  - URL & State Management
  - Touch & Mobile (44px targets, safe areas)
  - Light/Dark Mode
  - Anti-patterns Blocklist (11 items)
  - UI States Checklist
  - Red Flags
  - Rationalization Prevention
  - Output Format
  - Final Check

### A3: Create `brainstorming` skill

**Source:** `reference/cc10x-skills/brainstorming.md` (365 lines)
**Target:** `plugins/cc100x/skills/brainstorming/SKILL.md`
**Action:** Copy source to target, apply ALL mechanical name changes from table above.

**EXTRA CHANGES specific to brainstorming:**

1. **Line ~301 (Saving the Design → Step 2: Update Memory):**
   All `.claude/cc10x/activeContext.md` → `.claude/cc100x/activeContext.md`
   (3 occurrences in the memory update Edit examples)

2. **Line ~349 (After Brainstorming → option A):**
   `"use planning-patterns skill"` — ensure this reads `"use cc100x:planning-patterns skill"`

**Verify after creation:**
- No remaining occurrences of `cc10x` anywhere in the file
- Phase 1-4 preserved (Context, Explore, Approaches, Present)
- One Question at a Time methodology preserved
- YAGNI section preserved
- Spec File Workflow preserved
- Saving the Design (Two saves) preserved
- UI Mockup section preserved
- Rationalization Prevention preserved

---

## Phase B: Wire Skills into SKILL_HINTS Table [P0 — CRITICAL]

### B1: Update `plugins/cc100x/skills/cc100x-lead/SKILL.md`

**File:** `plugins/cc100x/skills/cc100x-lead/SKILL.md`
**Location:** Lines 569-581 (the SKILL_HINTS table)

**CURRENT table (lines 571-580):**
```
| Detected Pattern | Skill | Agents |
|------------------|-------|--------|
| **BUILD workflow** (any build/implement/create) | `cc100x:test-driven-development`, `cc100x:code-generation` | builder |
| **REVIEW workflow** (any review/audit/check) | `cc100x:code-review-patterns` | security-reviewer, performance-reviewer, quality-reviewer, live-reviewer |
| **DEBUG workflow** (any debug/fix/error) | `cc100x:debugging-patterns` | investigator |
| **PLAN workflow** (any plan/design/architect) | `cc100x:planning-patterns` | planner |
| **BUILD/DEBUG post-build** (hunter phase) | `cc100x:code-review-patterns` | hunter |
| External: new tech, unfamiliar library, complex integration | `cc100x:github-research` | planner, investigator |
| Debug exhausted: 3+ local attempts failed, external service error | `cc100x:github-research` | investigator |
| User explicitly requests: "research", "github", "octocode", "best practices" | `cc100x:github-research` | planner, investigator |
```

**REPLACE WITH (exact new table):**
```
| Detected Pattern | Skill | Agents |
|------------------|-------|--------|
| **BUILD workflow** (any build/implement/create) | `cc100x:test-driven-development`, `cc100x:code-generation`, `cc100x:architecture-patterns`, `cc100x:frontend-patterns` | builder |
| **REVIEW workflow** (any review/audit/check) | `cc100x:code-review-patterns`, `cc100x:architecture-patterns`, `cc100x:frontend-patterns` | security-reviewer, performance-reviewer, quality-reviewer, live-reviewer |
| **DEBUG workflow** (any debug/fix/error) | `cc100x:debugging-patterns`, `cc100x:architecture-patterns`, `cc100x:frontend-patterns` | investigator |
| **PLAN workflow** (any plan/design/architect) | `cc100x:planning-patterns`, `cc100x:brainstorming`, `cc100x:architecture-patterns`, `cc100x:frontend-patterns` | planner |
| **BUILD/DEBUG post-build** (hunter phase) | `cc100x:code-review-patterns`, `cc100x:architecture-patterns`, `cc100x:frontend-patterns` | hunter |
| **BUILD/DEBUG post-verification** (verifier phase) | `cc100x:debugging-patterns`, `cc100x:architecture-patterns`, `cc100x:frontend-patterns` | verifier |
| External: new tech, unfamiliar library, complex integration | `cc100x:github-research` | planner, investigator |
| Debug exhausted: 3+ local attempts failed, external service error | `cc100x:github-research` | investigator |
| User explicitly requests: "research", "github", "octocode", "best practices" | `cc100x:github-research` | planner, investigator |
```

**Changes explained:**
1. **BUILD:** Added `architecture-patterns` + `frontend-patterns` (CC10x component-builder had both)
2. **REVIEW:** Added `architecture-patterns` + `frontend-patterns` (CC10x code-reviewer had both)
3. **DEBUG:** Added `architecture-patterns` + `frontend-patterns` (CC10x bug-investigator had both)
4. **PLAN:** Added `brainstorming` + `architecture-patterns` + `frontend-patterns` (CC10x planner had all 3)
5. **post-build hunter:** Added `architecture-patterns` + `frontend-patterns` (CC10x hunter had both)
6. **NEW ROW: post-verification verifier:** Added `debugging-patterns` + `architecture-patterns` + `frontend-patterns` (CC10x verifier had all 3 — this was MISSING entirely)

---

## Phase C: Fix Agent Prompts [P1 — MAJOR]

### C1: Fix planner brainstorming reference

**File:** `plugins/cc100x/agents/planner.md`
**Location:** Line 60

**CURRENT:**
```
**If 3+ questions needed** → consider structured discovery to gather all requirements at once.
```

**REPLACE WITH:**
```
**If 3+ questions needed** → `Skill(skill="cc100x:brainstorming")` for structured discovery to gather all requirements at once.
```

### C2: Fix planner github-research reference

**File:** `plugins/cc100x/agents/planner.md`
**Location:** Lines 62-65

**CURRENT:**
```
## Conditional Research

- New/unfamiliar tech → web research via WebFetch
- Complex integration patterns → search GitHub for reference implementations
```

**REPLACE WITH:**
```
## Conditional Research

- New/unfamiliar tech → `Skill(skill="cc100x:github-research")` (falls back to WebFetch if unavailable)
- Complex integration patterns → `Skill(skill="cc100x:github-research")` for reference implementations
```

### C3: Fix investigator github-research reference

**File:** `plugins/cc100x/agents/investigator.md`
**Location:** Lines 36-39

**CURRENT:**
```
## Conditional Research

- External service/API bugs → web research via WebFetch
- 3+ local debugging attempts failed → external research
```

**REPLACE WITH:**
```
## Conditional Research

- External service/API bugs → `Skill(skill="cc100x:github-research")` (falls back to WebFetch if unavailable)
- 3+ local debugging attempts failed → `Skill(skill="cc100x:github-research")` for external research
```

---

## Phase D: Update Domain Skills Documentation [P1 — MAJOR]

### D1: Update `plugins/cc100x/CLAUDE.md` Domain Skills table

**File:** `plugins/cc100x/CLAUDE.md`
**Location:** Lines 30-43

**CURRENT:**
```
## Domain Skills (Loaded by CC100x Agents)

CC100x includes domain expertise skills that agents load automatically via SKILL_HINTS:

| Skill | Used By | Purpose |
|-------|---------|---------|
| `cc100x:debugging-patterns` | investigator, hunter, verifier | Systematic debugging, root cause tracing, LSP-powered analysis |
| `cc100x:test-driven-development` | builder | TDD Iron Law, Red-Green-Refactor, test quality |
| `cc100x:code-review-patterns` | security/performance/quality reviewers, live-reviewer, hunter | Two-stage review, security checklist, LSP analysis |
| `cc100x:planning-patterns` | planner | Plan structure, task granularity, risk assessment |
| `cc100x:code-generation` | builder | Universal questions, pattern matching, minimal code |
| `cc100x:github-research` | planner, investigator | External research via Octocode/GitHub, tiered fallbacks, checkpoint saves |

These are loaded automatically by the lead's SKILL_HINTS mechanism. You don't need to invoke them manually.
```

**REPLACE WITH:**
```
## Domain Skills (Loaded by CC100x Agents)

CC100x includes domain expertise skills that agents load automatically via SKILL_HINTS:

| Skill | Used By | Purpose |
|-------|---------|---------|
| `cc100x:debugging-patterns` | investigator, hunter, verifier | Systematic debugging, root cause tracing, LSP-powered analysis |
| `cc100x:test-driven-development` | builder | TDD Iron Law, Red-Green-Refactor, test quality |
| `cc100x:code-review-patterns` | security/performance/quality reviewers, live-reviewer, hunter | Two-stage review, security checklist, LSP analysis |
| `cc100x:planning-patterns` | planner | Plan structure, task granularity, risk assessment |
| `cc100x:code-generation` | builder | Universal questions, pattern matching, minimal code |
| `cc100x:github-research` | planner, investigator | External research via Octocode/GitHub, tiered fallbacks, checkpoint saves |
| `cc100x:architecture-patterns` | builder, all reviewers, investigator, hunter, verifier, planner | Functionality-first design, C4 views, API design, integration patterns, decision framework |
| `cc100x:frontend-patterns` | builder, all reviewers, investigator, hunter, verifier, planner | Loading states, accessibility (WCAG 2.1 AA), forms, animation, responsive design, component patterns |
| `cc100x:brainstorming` | planner | Structured discovery, one-question-at-a-time, YAGNI, incremental design validation |

These are loaded automatically by the lead's SKILL_HINTS mechanism. You don't need to invoke them manually.
```

---

## Phase E: Expand CC100x Bible [P1 — MAJOR]

### E1: Port missing sections from CC10x Bible

**File:** `docs/cc100x-bible.md`
**Source:** `reference/cc10x-orchestration-bible.md`

Add these sections (adapted for Agent Teams) after the existing content and before `## Reference Materials`:

**Sections to add (port from CC10x Bible with Agent Teams adaptations):**

1. **Glossary (CC100x Terms)** — Port from CC10x lines 17-26, update terms:
   - Router → Lead (the execution engine defined by `plugins/cc100x/skills/cc100x-lead/SKILL.md`)
   - Agents → 9 CC100x agents (list all: builder, security-reviewer, performance-reviewer, quality-reviewer, live-reviewer, hunter, verifier, investigator, planner)
   - Skills → 15 skills in `plugins/cc100x/skills/*/SKILL.md` (list all)
   - Memory → `.claude/cc100x/{activeContext.md, patterns.md, progress.md}`
   - Add: Teammate, Team Lead, Task List, Mailbox (Agent Teams terms)

2. **Skills vs Agents (Claude Code + Agent Teams Concepts)** — Port from CC10x lines 29-108, add Agent Teams differences:
   - Skills: same definition (Markdown files that instruct)
   - Agents: now Agent Teams teammates (own context window, can message each other, CAN see CLAUDE.md)
   - Key difference: CC10x agents used `context: fork` (cannot see CLAUDE.md), CC100x teammates CAN see project context
   - SKILL_HINTS still needed for conditional skills and user-global skills

3. **Orchestration Invariants** — Port from CC10x lines 116-126, update:
   - "Router is the ONLY entry point" → "Lead is the ONLY entry point"
   - Add: "Lead operates in delegate mode (coordination only, no code implementation)"
   - Add: "No two teammates edit the same file"
   - Add: "Teammates send Router Contracts; lead validates and persists"

4. **Agent Chain Protocols (CC100x Workflows)** — Port from CC10x lines 219-243, update for 4 workflows:
   - BUILD: builder → (live-reviewer concurrent) → hunter → verifier → memory
   - DEBUG: investigators (parallel) → debate → builder fix → quality-reviewer → hunter → verifier → memory
   - REVIEW: 3 reviewers (parallel) → challenge round → consensus → memory
   - PLAN: planner → memory (unchanged)

5. **Task Types and Prefixes** — Port from CC10x lines 466-474, update prefix to `CC100X`

6. **Non-Optional Behaviors** — Port from CC10x lines 566-572, add:
   - "Lead never implements code (delegate mode)"
   - "No two teammates edit the same file"

7. **Skill Loading Hierarchy (Definitive)** — Port from CC10x lines 340-370, update with CC100x SKILL_HINTS table and all 9 agents. Show which skills each agent gets via frontmatter + SKILL_HINTS.

**IMPORTANT:** Update the Architecture Components table (line 96-103) which currently says `teammates/*.md` and `protocols/*.md` — these paths are wrong. The correct paths are:
- Agents: `agents/*.md` (not `teammates/`)
- Protocols: Encoded in workflow skills (`skills/pair-build/`, `skills/review-arena/`, `skills/bug-court/`)

---

## Phase F: Verification Checklist [After All Phases] — ALL PASSED ✓

Verification completed 2026-02-06. All checks pass:

- [x] `plugins/cc100x/skills/architecture-patterns/SKILL.md` exists (360 lines)
- [x] `plugins/cc100x/skills/frontend-patterns/SKILL.md` exists (582 lines)
- [x] `plugins/cc100x/skills/brainstorming/SKILL.md` exists (364 lines)
- [x] No occurrence of `cc10x` in any of the 3 new skill files (verified via grep)
- [x] SKILL_HINTS table in lead has 9 rows (was 8, added verifier row)
- [x] All 6 existing workflow rows now include `architecture-patterns` + `frontend-patterns`
- [x] PLAN row includes `brainstorming`
- [x] New verifier row includes `debugging-patterns` + `architecture-patterns` + `frontend-patterns`
- [x] Planner line 60 references `Skill(skill="cc100x:brainstorming")`
- [x] Planner lines 64-65 reference `Skill(skill="cc100x:github-research")`
- [x] Investigator lines 38-39 reference `Skill(skill="cc100x:github-research")`
- [x] CLAUDE.md Domain Skills table has 9 rows (was 6, added 3)
- [x] CC100x Bible has Glossary, Skills vs Agents, Invariants, Chain Protocols, Task Types, Non-Optional Behaviors, Skill Loading Hierarchy, Task State Transitions sections
- [x] CC100x Bible Architecture Components table references correct paths (`agents/*.md`)
- [x] Total CC100x skills: 16 (verified via glob — 16 SKILL.md files under plugins/cc100x/skills/)

### Skill Count Verification

**CC100x Skills (final = 16):**
1. cc100x-lead (orchestration)
2. session-memory
3. router-contract
4. verification
5. review-arena (workflow)
6. bug-court (workflow)
7. pair-build (workflow)
8. code-review-patterns (domain)
9. debugging-patterns (domain)
10. test-driven-development (domain)
11. code-generation (domain)
12. planning-patterns (domain)
13. github-research (domain)
14. **architecture-patterns** (domain) ← NEW
15. **frontend-patterns** (domain) ← NEW
16. **brainstorming** (domain) ← NEW

**CC10x had 12 skills. CC100x now has 16 (12 ported + 4 new workflow skills).**
All 12 CC10x skills are now represented.

### Agent-to-Skill Mapping (Final — Must Match CC10x Coverage)

| Agent | Frontmatter Skills | SKILL_HINTS Skills | CC10x Equivalent Had |
|-------|-------------------|-------------------|---------------------|
| builder | session-memory, router-contract, verification | TDD, code-gen, architecture-patterns, frontend-patterns | session-memory, TDD, code-gen, verification, frontend-patterns, architecture-patterns ✓ |
| security-reviewer | router-contract, verification | code-review-patterns, architecture-patterns, frontend-patterns | code-review-patterns, verification, frontend-patterns, architecture-patterns ✓ |
| performance-reviewer | router-contract, verification | code-review-patterns, architecture-patterns, frontend-patterns | (same as above) ✓ |
| quality-reviewer | router-contract, verification | code-review-patterns, architecture-patterns, frontend-patterns | (same as above) ✓ |
| live-reviewer | router-contract, verification | code-review-patterns, architecture-patterns, frontend-patterns | N/A (new) |
| hunter | router-contract, verification | code-review-patterns, architecture-patterns, frontend-patterns | code-review-patterns, verification, frontend-patterns, architecture-patterns ✓ |
| verifier | router-contract, verification | debugging-patterns, architecture-patterns, frontend-patterns | architecture-patterns, debugging-patterns, verification, frontend-patterns ✓ |
| investigator | router-contract, verification | debugging-patterns, architecture-patterns, frontend-patterns, github-research (conditional) | session-memory, debugging-patterns, TDD, verification, architecture-patterns, frontend-patterns ✓* |
| planner | session-memory, router-contract, verification | planning-patterns, brainstorming, architecture-patterns, frontend-patterns, github-research (conditional) | session-memory, planning-patterns, architecture-patterns, brainstorming, frontend-patterns ✓ |

*Note: CC10x investigator had session-memory + TDD because it was a WRITE agent that implemented fixes. CC100x investigator is READ-ONLY (Bug Court design). Builder has session-memory + TDD and implements the fix. This is a deliberate architectural change, not a loss.

---

## Execution Order

1. **Phase A** (create 3 skills) — can be done in parallel (A1, A2, A3 are independent)
2. **Phase B** (wire SKILL_HINTS) — depends on Phase A (skills must exist before wiring)
3. **Phase C** (fix agent prompts) — depends on Phase A (brainstorming skill must exist)
4. **Phase D** (update docs) — depends on Phase B (table must be final)
5. **Phase E** (expand bible) — independent, can run with Phase C/D
6. **Phase F** (verify) — depends on ALL above
