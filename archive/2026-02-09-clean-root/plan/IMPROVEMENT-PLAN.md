# CC100x Master Improvement Plan

> **Created:** 2026-02-06
> **Status:** In Progress
> **Source:** 5 exhaustive audit reports comparing CC100x against CC10x originals
> **Goal:** Port ALL battle-tested CC10x content, adapt ONLY what Agent Teams requires changing, add Agent Teams-specific improvements

---

## Guiding Principles

1. **Port faithfully, adapt minimally.** Every CC10x line exists for a reason. Copy first, then adapt only what Agent Teams changes.
2. **Agent Teams additions are ADDITIVE.** New patterns (peer messaging, adversarial review, competing hypotheses) augment CC10x patterns — they never replace operational safeguards.
3. **Concrete > Abstract.** Always keep examples, code blocks, tables. Never replace with prose summaries.
4. **Every gate must survive.** Gates, checks, and enforcement statements from CC10x must be present in CC100x.
5. **Domain skills must be loaded.** Every agent needs its domain-specific skills in frontmatter.

---

## Priority Tiers

### Tier 1: CRITICAL (Blocks quality guarantee)
Files that must be fixed before CC100x can be trusted.

### Tier 2: HIGH (Major functional gaps)
Files with significant missing operational content.

### Tier 3: MEDIUM (Polish and completeness)
Files that work but are missing depth, examples, or fallback paths.

---

## File-by-File Improvement Roadmap

### FILE 1: `plugins/cc100x/skills/cc100x-lead/SKILL.md`
**Priority:** Tier 1 — CRITICAL
**Current:** 834 lines | **Was:** 535 lines | **Target:** ~800+ ✅
**Gaps:** ALL ADDRESSED

| # | What to Port/Add | Source | Status |
|---|-----------------|--------|--------|
| 1 | Add Circuit Breaker (count REM-FIX tasks, if >=3 → AskUserQuestion with 4 options) | cc10x-router lines 411-418 | ✅ DONE |
| 2 | Add Remediation Re-Review Loop (after ANY REM-FIX completes → re-review + re-hunt before verifier) | cc10x-router lines 456-477, Bible lines 476-505 | ✅ DONE |
| 3 | Add 3+ debug failure research trigger (count `[DEBUG-N]:` lines, if 3+ → external research) | cc10x-router lines 273-333 | ✅ DONE |
| 4 | Add Memory update safety rules (6 explicit rules including STOP-and-retry) | cc10x-router lines 97-106 | ✅ DONE |
| 5 | Add Task Dependency Safety rules (forward-only, cycle detection, skip-and-log) | cc10x-router lines 133-145 | ✅ DONE |
| 6 | Fill ALL task description placeholders (replace every `...` with actual instructions) | cc10x-router lines 162-253 | ✅ DONE |
| 7 | Add REM-EVIDENCE task creation (task-enforced, not chat-request) | cc10x-router lines 388-394 | ✅ DONE |
| 8 | Add REM-FIX with REMEDIATION_REASON from contract | cc10x-router lines 419-430 | ✅ DONE |
| 9 | Add Debug Attempt Counting format (`[DEBUG-N]: {tried} → {result}`) | cc10x-router lines 281-289 | ✅ DONE |
| 10 | Add External service error trigger | cc10x-router lines 273-278 | ✅ DONE |
| 11 | Add THREE-PHASE research pattern to DEBUG workflow | cc10x-router lines 322-333 | ✅ DONE |
| 12 | Add Skill loading hierarchy / SKILL_HINTS mechanism | cc10x-router lines 496-509 | ✅ DONE |
| 13 | Add Agent invocation template (structured context) | cc10x-router lines 335-374 | ✅ DONE |
| 14 | Add missing gates (RESEARCH_EXECUTED, RESEARCH_PERSISTED, REQUIREMENTS_CLARIFIED, TASKS_CREATED) | cc10x-router gates list | ✅ DONE |
| 15 | Add Results collection pattern (how findings pass downstream) | cc10x-router lines 628-674 | ✅ DONE |
| 16 | Add Memory anchor integrity rule ("never break anchors") | cc10x Bible line 291-292 | ✅ DONE |
| 17 | Add cross-project collision safety | cc10x-router lines 121-123 | ✅ DONE |
| 18 | Add E2E verification to DEBUG chain | cc10x-router DEBUG chain | ✅ DONE |
| 19 | Add BUILD chain task-enforced dependencies | cc10x-router BUILD chain | ✅ DONE |
| 20 | Add compact Agent Chain summary table | New addition for readability | ✅ DONE |

---

### FILE 2: `plugins/cc100x/skills/session-memory/SKILL.md`
**Priority:** Tier 1 — CRITICAL
**Current:** 556 lines | **Was:** 274 lines | **Target:** ~500+ ✅
**Gaps:** ALL ADDRESSED

| # | What to Port/Add | Source | Status |
|---|-----------------|--------|--------|
| 1 | Port entire READ Triggers section (4 subsections, file selection matrix) | cc10x session-memory lines 354-401 | ✅ DONE |
| 2 | Port Decision Integration protocol (3-question gate) | cc10x session-memory lines 403-418 | ✅ DONE |
| 3 | Port "Use Read Tool, NOT Bash(cat)" section (2 anti-pattern code blocks) | cc10x session-memory lines 82-112 | ✅ DONE |
| 4 | Port Rationalization Prevention table (4 excuse/reality pairs) | cc10x session-memory lines 539-546 | ✅ DONE |
| 5 | Port Edit Failure Recovery procedure | cc10x session-memory lines 190-192 | ✅ DONE |
| 6 | Port "Why This Matters" section (quote + bullet list) | cc10x session-memory lines 114-125 | ✅ DONE |
| 7 | Port "When Learning Patterns" recipe | cc10x session-memory lines 484-496 | ✅ DONE |
| 8 | Port "When Completing Tasks" recipe (Options A + B) | cc10x session-memory lines 498-513 | ✅ DONE |
| 9 | Port "Integration with Agents" fallback protocol | cc10x session-memory lines 515-527 | ✅ DONE |
| 10 | Port READ/WRITE Side explanation block | cc10x session-memory lines 50-58 | ✅ DONE |
| 11 | Port Git context gathering at workflow start | cc10x session-memory lines 437-440 | ✅ DONE |
| 12 | Port "NEVER use as anchors" warning | cc10x session-memory lines 311-314 | ✅ DONE |
| 13 | Port Canonical section insertion rule | cc10x session-memory lines 186-188 | ✅ DONE |
| 14 | Port Checkpoint Pattern code block + "when in doubt, update NOW" | cc10x session-memory lines 162-168 | ✅ DONE |
| 15 | Port Workflow END examples B + C + "WHY Edit not Write" | cc10x session-memory lines 455-480 | ✅ DONE |
| 16 | Port "If file doesn't exist" fresh-start guidance | cc10x session-memory line 449 | ✅ DONE |
| 17 | Port debug annotation format `[DEBUG-N]` in activeContext template | cc10x session-memory line 207 | ✅ DONE |
| 18 | Port "Merged sections" explanations for both templates | cc10x session-memory lines 230-233, 294-297 | ✅ DONE |
| 19 | Restore Tasks surface "not guaranteed" warning | cc10x session-memory lines 38-40 | ✅ DONE |
| 20 | Restore concurrency "if you must" escape hatch | cc10x session-memory lines 150-152 | ✅ DONE |
| 21 | Port Research-sourced gotcha pattern with source attribution | cc10x session-memory line 257 | ✅ DONE |

---

### FILE 3: `plugins/cc100x/skills/verification/SKILL.md`
**Priority:** Tier 1 — CRITICAL
**Current:** 398 lines | **Was:** 153 lines | **Target:** ~380+ ✅
**Gaps:** ALL ADDRESSED

| # | What to Port/Add | Source | Status |
|---|-----------------|--------|--------|
| 1 | Port Overview section with anti-loophole clause | cc10x verification lines 9-15 | ✅ DONE |
| 2 | Port "Agent completed" Common Failures row | cc10x verification line 51 | ✅ DONE |
| 3 | Port Key Patterns section (5 categories with ✅/❌) | cc10x verification lines 84-114 | ✅ DONE |
| 4 | Port 5 missing Rationalization Prevention rows | cc10x verification lines 76-82 | ✅ DONE |
| 5 | Port 2 missing Red Flag bullets + parenthetical examples | cc10x verification lines 59-66 | ✅ DONE |
| 6 | Port "When To Apply" scope definition | cc10x verification lines 126-143 | ✅ DONE |
| 7 | Port "Why This Matters" (real failure stories) | cc10x verification lines 116-124 | ✅ DONE |
| 8 | Port Self-Critique Verdict (PROCEED YES/NO gate) | cc10x verification lines 162-168 | ✅ DONE |
| 9 | Port missing Self-Critique items (commented-out code, unexpected files) | cc10x verification lines 156, 163 | ✅ DONE |
| 10 | Port Verification Checklist (7 annotated items) | cc10x verification lines 185-196 | ✅ DONE |
| 11 | Port Output Criteria + Deviations from Plan sections | cc10x verification lines 197-223 | ✅ DONE |
| 12 | Port Goal-Backward Quick Check Template | cc10x verification lines 243-261 | ✅ DONE |
| 13 | Port Goal-Backward "When to Apply" + Iron Law reinforcement | cc10x verification lines 263-268 | ✅ DONE |
| 14 | Port React Component Stubs table | cc10x verification lines 284-290 | ✅ DONE |
| 15 | Port API Route Stubs table | cc10x verification lines 292-298 | ✅ DONE |
| 16 | Port Function Stubs table | cc10x verification lines 300-306 | ✅ DONE |
| 17 | Port Wiring verification deep checks (response usage, result return) | cc10x verification lines 324-333 | ✅ DONE |
| 18 | Port Wiring Red Flags table | cc10x verification lines 335-341 | ✅ DONE |
| 19 | Port Line Count Minimums table | cc10x verification lines 343-348 | ✅ DONE |
| 20 | Port Export/Import Verification subsection | cc10x verification lines 349-372 | ✅ DONE |
| 21 | Port Auth Protection Verification subsection | cc10x verification lines 374-390 | ✅ DONE |

---

### FILE 4: `plugins/cc100x/skills/review-arena/SKILL.md`
**Priority:** Tier 2 — HIGH
**Current:** 327 lines | **Was:** 142 lines | **Target:** ~300+ ✅
**Gaps:** ALL ADDRESSED

| # | What to Port/Add | Source | Status |
|---|-----------------|--------|--------|
| 1 | Add Two-Stage Review (spec compliance BEFORE quality) | cc10x code-review-patterns | ✅ DONE |
| 2 | Add Signal Quality Rule + "Do NOT Flag" list | cc10x code-review-patterns | ✅ DONE |
| 3 | Add Security Quick-Scan Commands | cc10x code-review-patterns | ✅ DONE |
| 4 | Add individual reviewer output format | New design needed | ✅ DONE |
| 5 | Add LSP-Powered analysis toolkit | cc10x code-review-patterns | ✅ DONE |
| 6 | Add Rationalization Prevention table (review-specific) | cc10x code-review-patterns | ✅ DONE |
| 7 | Add Red Flags list | cc10x code-review-patterns | ✅ DONE |
| 8 | Embed fresh evidence mandate from verification skill | cc10x verification-before-completion | ✅ DONE |
| 9 | Embed memory concurrency rule | cc10x session-memory | ✅ DONE |
| 10 | Add reviewer timeout/failure handling | New design needed | ✅ DONE |
| 11 | Add Severity classification (4 levels: CRITICAL/MAJOR/MINOR/NIT) | cc10x code-review-patterns | ✅ DONE |
| 12 | Add Self-Critique Gate reference | cc10x verification-before-completion | ✅ DONE |

---

### FILE 5: `plugins/cc100x/skills/bug-court/SKILL.md`
**Priority:** Tier 2 — HIGH (includes 1 architectural fix)
**Current:** 401 lines | **Was:** 167 lines | **Target:** ~380+ ✅
**Gaps:** ALL ADDRESSED

| # | What to Port/Add | Source | Status |
|---|-----------------|--------|--------|
| 1 | **FIX: Make investigators READ-ONLY** (architectural fix for file isolation) | Agent Teams limitation | ✅ DONE |
| 2 | Add Common Debugging Scenarios (6 scenario-specific toolkits) | cc10x debugging-patterns | ✅ DONE |
| 3 | Add Root Cause Tracing Technique (5-step) | cc10x debugging-patterns | ✅ DONE |
| 4 | Add LSP-Powered Root Cause Tracing (5-step) | cc10x debugging-patterns | ✅ DONE |
| 5 | Add Cognitive Biases table (confirmation, anchoring, availability, sunk cost) | cc10x debugging-patterns | ✅ DONE |
| 6 | Add "When to Restart Investigation" (5 criteria + restart protocol) | cc10x debugging-patterns | ✅ DONE |
| 7 | Add Red Flags list (11 debugging red flags) | cc10x debugging-patterns | ✅ DONE |
| 8 | Add Rationalization Prevention table (8 debugging excuses) | cc10x debugging-patterns | ✅ DONE |
| 9 | Embed fresh evidence mandate | cc10x verification-before-completion | ✅ DONE |
| 10 | Embed memory concurrency rule | cc10x session-memory | ✅ DONE |
| 11 | Add investigator timeout/failure handling | New design needed | ✅ DONE |
| 12 | Add file conflict resolution mechanism | Agent Teams adaptation | ✅ DONE |
| 13 | Add Context Retrieval 3-cycle pattern | cc10x debugging-patterns | ✅ DONE |
| 14 | Add "When Process Reveals No Root Cause" guidance | cc10x debugging-patterns | ✅ DONE |

---

### FILE 6: `plugins/cc100x/skills/pair-build/SKILL.md`
**Priority:** Tier 2 — HIGH
**Current:** 350 lines | **Was:** 155 lines | **Target:** ~330+ ✅
**Gaps:** ALL ADDRESSED

| # | What to Port/Add | Source | Status |
|---|-----------------|--------|--------|
| 1 | Add TDD Iron Law with enforcement ("Delete it. Start over.") | cc10x test-driven-development | ✅ DONE |
| 2 | Add Verify RED mandate (MANDATORY, never skip) | cc10x test-driven-development | ✅ DONE |
| 3 | Add Verify GREEN mandate (pass + no regressions) | cc10x test-driven-development | ✅ DONE |
| 4 | Add Good Tests criteria table | cc10x test-driven-development | ✅ DONE |
| 5 | Add TDD Red Flags (13 rationalizations) | cc10x test-driven-development | ✅ DONE |
| 6 | Add TDD Rationalization Prevention (11 excuses) | cc10x test-driven-development | ✅ DONE |
| 7 | Add TDD Verification Checklist (8 items) | cc10x test-driven-development | ✅ DONE |
| 8 | Add Self-Critique Gate reference | cc10x verification-before-completion | ✅ DONE |
| 9 | Add Goal-Backward Lens reference (after verifier, before marking BUILD done) | cc10x verification-before-completion | ✅ DONE |
| 10 | Add Stub Detection patterns reference | cc10x verification-before-completion | ✅ DONE |
| 11 | Add builder-reviewer synchronization pattern (polling convention) | Agent Teams adaptation | ✅ DONE |
| 12 | Embed fresh evidence mandate | cc10x verification-before-completion | ✅ DONE |
| 13 | Add remediation loop circuit breaker reference | cc10x-router | ✅ DONE |
| 14 | Add Mocking External Dependencies rules | cc10x test-driven-development | ✅ DONE |
| 15 | Add Refactor discipline ("After green only") | cc10x test-driven-development | ✅ DONE |

---

### FILE 7: `plugins/cc100x/skills/router-contract/SKILL.md`
**Priority:** Tier 2 — HIGH
**Current:** 279 lines | **Was:** 207 lines | **Target:** ~270+ ✅
**Gaps:** ALL ADDRESSED

| # | What to Port/Add | Source | Status |
|---|-----------------|--------|--------|
| 1 | Add 4-level severity classification (CRITICAL/MAJOR/MINOR/NIT) | cc10x code-review-patterns | ✅ DONE |
| 2 | Add SPEC_COMPLIANCE tracking field | cc10x code-review-patterns Two-Stage | ✅ DONE |
| 3 | Add TIMESTAMP field | New design | ✅ DONE |
| 4 | Add AGENT_ID field (for Bug Court multi-investigator) | New design | ✅ DONE |
| 5 | Add FILES_MODIFIED field (for builder/investigator) | New design | ✅ DONE |
| 6 | Add DEVIATIONS_FROM_PLAN field | cc10x verification | ✅ DONE |
| 7 | Add malformed contract retry logic (max retries, fallback) | New design | ✅ DONE |
| 8 | Define "abort" behavior in circuit breaker (revert, cleanup, persist) | New design | ✅ DONE |

---

### FILES 8-16: All 9 Agent Definitions
**Priority:** Tier 2 — HIGH (cross-cutting fixes)

Each agent needs the same cross-cutting fixes:

| # | What to Port/Add | Applies To | Status |
|---|-----------------|-----------|--------|
| 1 | Add missing domain skills to frontmatter | ALL 9 agents | ✅ DONE |
| 2 | Add Router Handoff section (human-readable extraction) | ALL 9 agents | ✅ DONE |
| 3 | Add TaskCreate pattern for follow-up issues | ALL 9 agents | ✅ DONE |
| 4 | Add "Findings" section to output | ALL 9 agents | ✅ DONE |
| 5 | Add "Follow-up tasks created" line | ALL 9 agents | ✅ DONE |
| 6 | Add Dev Journal "Assumptions Made" section | ALL 9 agents | ✅ DONE |
| 7 | Add Dev Journal "What's Next" narrative | ALL 9 agents | ✅ DONE |
| 8 | Add Memory "Why" explanation | ALL 9 agents | ✅ DONE |
| 9 | Add SKILL_HINTS failure handling | ALL 9 agents | ✅ DONE |
| 10 | Add Task Completion section | ALL 9 agents | ✅ DONE |

Agent-specific fixes:

| Agent | Fix | Status |
|-------|-----|--------|
| builder | Add TDD gate statement, Plan gate, Verify functionality step | ✅ DONE |
| security-reviewer | Add Git context, full 5-point auth flow verification | ✅ DONE |
| performance-reviewer | Add Git context | ✅ DONE |
| quality-reviewer | Add Git context | ✅ DONE |
| hunter | Add CRITICAL remediation enforcement, Severity rubric examples | ✅ DONE |
| verifier | Add Rollback Decision detail, Decision+Rationale template | ✅ DONE |
| investigator | Add Anti-Hardcode Gate variant descriptions, Conditional Research, Context Retrieval 3-cycle, Variant Scan step, Assumptions+Confidence output | ✅ DONE |
| planner | Add AskUserQuestion example, "3+ questions" brainstorming trigger, Conditional Research, Context Retrieval 3-cycle, Confidence Score factors, SKILL_HINTS for BUILD output | ✅ DONE |
| live-reviewer | Add Git context (diff of specific file), confidence scoring | ✅ DONE |

---

### FILE 17: `plugins/cc100x/CLAUDE.md`
**Priority:** Tier 3 — MEDIUM

| # | What to Update | Status |
|---|---------------|--------|
| 1 | Update skill references after rewrites | ✅ DONE |
| 2 | Add domain skill loading instructions (debugging-patterns, tdd, etc.) | ✅ DONE |

---

## Domain Skills to Create

CC10x had 12 skills. CC100x currently has 7. The missing 5 domain skills need to be created as CC100x-adapted versions:

| Skill | CC10x Source | Used By | Status |
|-------|-------------|---------|--------|
| `debugging-patterns` | reference/cc10x-skills/debugging-patterns.md | investigator, hunter, verifier | ✅ DONE |
| `test-driven-development` | reference/cc10x-skills/test-driven-development.md | builder, investigator | ✅ DONE |
| `code-review-patterns` | reference/cc10x-skills/code-review-patterns.md | all 3 reviewers, live-reviewer, hunter | ✅ DONE |
| `planning-patterns` | reference/cc10x-skills/planning-patterns.md | planner | ✅ DONE |
| `code-generation` | reference/cc10x-skills/code-generation.md | builder | ✅ DONE |

Optional (load from user's CLAUDE.md Complementary Skills if present):
- `architecture-patterns` — Used across many agents
- `frontend-patterns` — Used across many agents
- `brainstorming` — Used by planner

---

## Agent Teams Alignment Items (Added Post-Audit)

Items discovered by cross-referencing against `reference/agent-teams-complete-docs.md` and `reference/anthropic-2026-features-research.md`:

### Lead Skill — Additional Items

| # | What to Add | Source | Status |
|---|------------|--------|--------|
| 21 | Add session interruption recovery protocol (Agent Teams: `/resume` doesn't restore teammates — lead must detect and re-spawn) | agent-teams-complete-docs line 320 | ✅ DONE |
| 22 | Add task status lag nudge protocol (teammates sometimes forget to mark tasks completed — lead must check and nudge) | agent-teams-complete-docs line 321 | ✅ DONE |
| 23 | Add "no nested teams" constraint to Bug Court (Phase 6 "abbreviated Review Arena" must reuse existing team, not spawn new team) | agent-teams-complete-docs line 324 | ✅ DONE |
| 24 | Leverage Plan Approval Mode for planner agent (teammate works in read-only plan mode until lead approves) | agent-teams-complete-docs lines 142-158 | ✅ DONE |
| 25 | Add task sizing guidance (5-6 tasks per teammate, self-contained deliverables) | agent-teams-complete-docs line 253 | ✅ DONE |
| 26 | Document SKILL_HINTS model change: Agent Teams teammates CAN see CLAUDE.md + project skills (unlike CC10x subagents). SKILL_HINTS still needed for conditional/user-global skills only. | agent-teams-complete-docs line 199, 329 | ✅ DONE |
| 27 | Add delegate mode enforcement instruction ("Lead MUST use delegate mode — press Shift+Tab after team creation") | agent-teams-complete-docs lines 160-166 | ✅ DONE |

### Workflow Skills — Additional Items

| # | What to Add | Skill | Source | Status |
|---|------------|-------|--------|--------|
| W1 | Bug Court: Define no-nested-teams workaround for post-fix Review Arena (reuse team, spawn reviewer teammates into existing team) | bug-court | agent-teams-complete-docs line 324 | ✅ DONE |
| W2 | Pair Build: Define message-based synchronization pattern (builder sends review request, polls for incoming messages before next module; timeout after N iterations) | pair-build | agent-teams-complete-docs line 265 (no sync primitives) | ✅ DONE |
| W3 | All workflows: Add session resumption recovery (if session interrupted mid-workflow, lead detects missing teammates via team config, re-spawns with context from task descriptions + memory files) | all | agent-teams-complete-docs line 320 | ✅ DONE (in cc100x-lead) |
| W4 | All workflows: Add pre-compaction memory checkpoint trigger for long-running teammates (Bug Court multi-round, Pair Build multi-module) | all | anthropic-2026-features lines 625-633 (context compaction) | ✅ DONE (in session-memory) |

### Tasks System — Additional Items

| # | What to Add | Source | Status |
|---|------------|--------|--------|
| T1 | Document Tasks system capabilities (DAGs via blockedBy/blocks, filesystem persistence at `~/.claude/tasks`, cross-session sharing via `CLAUDE_CODE_TASK_LIST_ID`) | anthropic-2026-features lines 440-466 | ✅ DONE |
| T2 | Add CLAUDE_CODE_TASK_LIST_ID awareness to lead skill (task lists can be shared across sessions, scope before resuming) | anthropic-2026-features line 461, cc10x-router line 128 | ✅ DONE |
| T3 | Add task deletion cleanup protocol (use TaskUpdate with `deleted` status for abandoned/obsolete tasks) | anthropic-2026-features line 463 | ✅ DONE |

### Future Enhancements (Not Blocking, But Worth Tracking)

| # | Enhancement | Source | Status |
|---|-----------|--------|--------|
| F1 | Explore Hooks system for verification enforcement (PostToolUse hook could catch unverified completion claims) | anthropic-2026-features lines 645-656 | ⬜ FUTURE |
| F2 | Explore Background Agents for parallel research (`&` suffix for non-blocking execution) | anthropic-2026-features lines 601-608 | ⬜ FUTURE |
| F3 | Explore MCP Tool Search implications (server instructions field critical for discoverability when using MCP tools) | anthropic-2026-features lines 471-497 | ⬜ FUTURE |

---

## Execution Order

### Phase A: Core Infrastructure (Tier 1 — CRITICAL)
These files form the foundation. Fix them first.

1. **session-memory/SKILL.md** — Memory protocol is used by everything
2. **verification/SKILL.md** — Verification gates are referenced by everything
3. **cc100x-lead/SKILL.md** — The orchestration brain

### Phase B: Workflow Skills (Tier 2 — HIGH)
These define the 4 workflows.

4. **review-arena/SKILL.md** — Used in BUILD and after DEBUG fixes
5. **bug-court/SKILL.md** — Includes architectural fix (READ-ONLY investigators)
6. **pair-build/SKILL.md** — Used for BUILD workflow
7. **router-contract/SKILL.md** — Contract schema updates

### Phase C: Agent Definitions (Tier 2 — HIGH)
Cross-cutting fixes applied to all 9 agents.

8. **All 9 agent .md files** — Frontmatter skills, Router Handoff, TaskCreate, etc.

### Phase D: Domain Skills (Tier 2 — HIGH)
Create the 5 missing domain skills.

9. **debugging-patterns, test-driven-development, code-review-patterns, planning-patterns, code-generation**

### Phase E: Polish (Tier 3 — MEDIUM)
10. **CLAUDE.md** updates, README updates, final integration testing

---

## Tracking Legend

| Symbol | Meaning |
|--------|---------|
| ⬜ TODO | Not started |
| 🔄 IN PROGRESS | Currently being worked on |
| ✅ DONE | Completed and verified |
| ❌ BLOCKED | Blocked by dependency |
| ⏭️ SKIPPED | Intentionally skipped with rationale |

---

## Change Log

| Date | Phase | Files Changed | Summary |
|------|-------|--------------|---------|
| 2026-02-06 | Research | plan/RESEARCH.md, plan/IMPROVEMENT-PLAN.md | Initial audit results compiled, improvement plan created |
| 2026-02-06 | Phase A | session-memory/SKILL.md | Rewritten: 274→556 lines. All 21 items ported from CC10x. |
| 2026-02-06 | Phase A | verification/SKILL.md | Rewritten: 153→398 lines. All 21 items ported from CC10x. |
| 2026-02-06 | Phase A | cc100x-lead/SKILL.md | Rewritten: 535→834 lines. All 20 items + 6/7 Agent Teams alignment items ported. |
| 2026-02-06 | Phase B | review-arena/SKILL.md | Rewritten: 142→327 lines. All 12 items ported. Two-Stage Review, Signal Quality, LSP, Security Quick-Scan added. |
| 2026-02-06 | Phase B | bug-court/SKILL.md | Rewritten: 167→401 lines. All 14 items ported. Investigators fixed to READ-ONLY. Root Cause Tracing, LSP, 6 debugging scenarios, Cognitive Biases added. |
| 2026-02-06 | Phase B | pair-build/SKILL.md | Rewritten: 155→350 lines. All 15 items ported. TDD Iron Law, RED/GREEN mandates, builder-reviewer sync, verification checklist added. |
| 2026-02-06 | Phase B | router-contract/SKILL.md | Rewritten: 207→279 lines. All 8 items ported. 5 new fields, severity classification, retry logic, abort behavior added. |
| 2026-02-06 | Phase C | All 9 agent .md files | Rewritten: ~1,240→1,912 lines total (+54%). 10 cross-cutting fixes + 9 agent-specific fixes. Investigator fixed to READ-ONLY. All agents have Router Contract, Router Handoff, TaskCreate, Memory Why, Task Completion sections. |
| 2026-02-06 | Phase D | 5 domain skill SKILL.md files | Created: debugging-patterns (536), test-driven-development (470), code-review-patterns (387), planning-patterns (509), code-generation (324). Total 2,226 lines. Faithful port with cc10x→cc100x adaptations. |
| 2026-02-06 | Phase E | cc100x-lead/SKILL.md, CLAUDE.md | Plan Approval Mode for planner (#24). Tasks system docs (T1-T3): DAG capabilities, CLAUDE_CODE_TASK_LIST_ID cross-session sharing, task deletion cleanup. CLAUDE.md updated with domain skill table. |
