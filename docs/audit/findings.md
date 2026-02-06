# CC10x → CC100x Audit Findings

## Legend
- **CRITICAL**: Functionality completely dropped or logic missing
- **MAJOR**: Significant logic/methodology not carried over
- **MINOR**: Small details, wording, or edge cases missing
- **OK**: Properly transitioned
- **ENHANCED**: CC100x improved upon CC10x

---

## Phase 1: Orchestration Core

### Files Compared:
- CC10x: `reference/cc10x-orchestration-bible.md` (605 lines), `reference/cc10x-router-SKILL.md` (674 lines), `reference/cc10x-orchestration-logic-analysis.md` (699 lines)
- CC100x: `docs/cc100x-bible.md` (114 lines), `plugins/cc100x/skills/cc100x-lead/SKILL.md` (924 lines)

### 1.1 Decision Tree — OK
CC100x lead has the identical 4-priority decision tree (ERROR→DEBUG, PLAN→PLAN, REVIEW→REVIEW, DEFAULT→BUILD) with same keywords and conflict resolution. No changes needed.

### 1.2 Memory Protocol — OK
Memory load (mkdir → Read 3 files), auto-heal validation, update rules (Edit not Write, Read-back verify), and template validation gate are all carried over verbatim. Path changed from `.claude/cc10x/` to `.claude/cc100x/`. Memory file structure (activeContext.md, patterns.md, progress.md) with all required sections preserved.

### 1.3 Task-Based Orchestration — OK / ENHANCED
All 4 workflow task hierarchies (BUILD, DEBUG, REVIEW, PLAN) are present. BUILD adapted to Pair Build (builder + live-reviewer instead of just builder). DEBUG adapted to Bug Court (3 investigators + debate). REVIEW adapted to Review Arena (3 specialized reviewers + challenge). PLAN unchanged. Memory Update task-enforced pattern preserved for all 4.

### 1.4 Chain Execution Loop — OK / ENHANCED
CC100x "Team Execution Loop" matches CC10x "Chain Execution Loop" logic: find runnable tasks → start agents → validate → next. Enhanced with Agent Teams specifics: delegate mode, teammate messaging, self-claim mode, team shutdown.

### 1.5 Router Contract Validation — OK
3-step validation (check contract → parse/validate → output evidence) preserved identically. Circuit breaker (3+ REM-FIX), BLOCKING/REQUIRES_REMEDIATION logic, conflict handling for parallel reviewers, MEMORY_NOTES collection — all present.

### 1.6 Remediation Re-Review Loop — OK
CC10x: `code-reviewer + silent-failure-hunter re-review → verifier`
CC100x: `quality-reviewer + hunter re-review → verifier`
Logic preserved, agent names updated for CC100x.

### 1.7 SKILL_HINTS / Skill Loading — MAJOR CHANGE (see 1.7a below)
CC10x used agent frontmatter `skills:` for automatic preloading of ALL CC10x internal skills. Only `github-research` was conditional.
CC100x uses SKILL_HINTS for workflow-based skill passing because Agent Teams teammates CAN see project skills.

**1.7a — FINDING [MAJOR]: CC10x skills loaded via agent frontmatter are now ALL passed as SKILL_HINTS in CC100x**
In CC10x, each agent had a specific `skills:` frontmatter listing (e.g., component-builder had: session-memory, test-driven-development, code-generation, verification-before-completion, frontend-patterns, architecture-patterns).
In CC100x, the lead passes skills per-workflow (BUILD gets TDD+code-gen, REVIEW gets code-review-patterns, etc.).
**MISSING SKILLS in SKILL_HINTS table:**
- `verification-before-completion` (now `cc100x:verification`) — NOT listed in CC100x SKILL_HINTS table for ANY workflow
- `architecture-patterns` — NO CC100x equivalent skill exists (see Phase 4)
- `frontend-patterns` — NO CC100x equivalent skill exists (see Phase 4)
- `brainstorming` — NO CC100x equivalent skill exists (see Phase 4)
- `session-memory` — NOT listed in CC100x SKILL_HINTS table. Only WRITE agents (builder, investigator, planner) loaded this in CC10x, but CC100x SKILL_HINTS table doesn't mention it.

### 1.8 Active Workflow Check — OK
Orphan check, legacy compatibility, task scope safety — all preserved.

### 1.9 TODO Task Handling — OK
Post-workflow TODO check preserved identically.

### 1.10 Gates Checklist — ENHANCED
CC10x had 9 gates. CC100x has 12 gates (adds TEAM_CREATED, CONTRACTS_VALIDATED, TEAM_SHUTDOWN). All original 9 are preserved.

### 1.11 Agent Invocation Template — OK
Template structure preserved. Updated terminology (router→lead, agent→teammate). SKILL_HINTS passthrough, memory notes instructions, task ID requirement — all present.

### 1.12 Results Collection Pattern — OK / ENHANCED
Adapted from 2-agent parallel (reviewer + hunter) to multi-agent parallel (3 reviewers, multiple investigators). Enhanced with peer messaging for challenge rounds.

### 1.13 CC100x Bible vs CC10x Bible — MAJOR (detail loss)
CC10x Bible: 605 lines of dense specification
CC100x Bible: 114 lines, mostly overview

**MISSING from CC100x Bible (may be OK if covered in lead SKILL.md):**
- Glossary section (Router, Workflow, Agents, Skills, Memory, Router Contract, Dev Journal definitions)
- Skills vs Agents explanation (Claude Code platform concepts)
- External Skill Conflict Risk design decision
- Agent Output Requirements / Router Contract format specification
- Router Contract by Agent table (STATUS values per agent, BLOCKING conditions)
- Task Types and Prefixes table (Workflow, Agent, REM-EVIDENCE, REM-FIX, TODO)
- Non-Optional Behaviors (hard rules list)
- Task State Transition diagram
- Skill Loading Hierarchy definitive table (per-agent mapping)

### 1.14 Orchestration Logic Analysis — NOT PORTED (detail patterns)
The CC10x logic analysis doc has detailed "English tricks" and patterns:

**FINDING [MINOR]: These CC10x logic analysis patterns have no CC100x equivalent doc:**
- English Tricks (permission-free ops, gate enforcement, confidence scoring, rationalization prevention tables, iron laws catalog)
- Agent-Specific Gates:
  - component-builder Plan File Check gate
  - bug-investigator Anti-Hardcode Gate (variant dimension checking)
- Context Retrieval Pattern (3-cycle explore pattern for planner/investigator)
- Hydration Pattern (Tasks = execution, Memory = persistence)
- Verification Flow with Goal-Backward Lens
- Plan → Build Handoff pattern
- 3-Task Rule (skip orchestration for <3 steps)

### 1.15 Session Interruption Recovery — ENHANCED (new for Agent Teams)
CC100x adds recovery protocol for /resume not restoring teammates. Not in CC10x (wasn't needed).

### 1.16 Agent Teams Constraints — ENHANCED (new)
CC100x adds 8 critical constraints specific to Agent Teams. New and appropriate.

### 1.17 Model Selection Guidance — ENHANCED (new)
CC100x adds per-teammate model recommendations (Opus vs Sonnet). New and appropriate.

### 1.18 Workflow Execution Steps — OK
All 4 workflows (BUILD, DEBUG, REVIEW, PLAN) have preserved steps. DEBUG retains debug attempt counting, research triggers, 3-phase research. BUILD retains Plan-First Gate. PLAN retains Plan Approval Mode (enhanced with Agent Teams plan_approval_request/response).

---

## Phase 2: Agents

### CRITICAL CROSS-CUTTING FINDING: Frontmatter Skill Stripping

**Every CC100x agent has dramatically fewer frontmatter skills than its CC10x counterpart.**

| Agent | CC10x Frontmatter Skills | CC100x Frontmatter Skills |
|-------|-------------------------|--------------------------|
| builder | session-memory, TDD, code-gen, verification, frontend-patterns, architecture-patterns (6) | session-memory, router-contract, verification (3) |
| security-reviewer | code-review-patterns, verification, frontend-patterns, architecture-patterns (4) | router-contract, verification (2) |
| performance-reviewer | (same as above) | router-contract, verification (2) |
| quality-reviewer | (same as above) | router-contract, verification (2) |
| hunter | code-review-patterns, verification, frontend-patterns, architecture-patterns (4) | router-contract, verification (2) |
| verifier | architecture-patterns, debugging-patterns, verification, frontend-patterns (4) | router-contract, verification (2) |
| investigator | session-memory, debugging-patterns, TDD, verification, architecture-patterns, frontend-patterns (6) | router-contract, verification (2) |
| planner | session-memory, planning-patterns, architecture-patterns, brainstorming, frontend-patterns (5) | session-memory, router-contract, verification (3) |

**Impact:** CC100x relies on SKILL_HINTS from lead to bridge this gap. But SKILL_HINTS only covers: TDD, code-gen, code-review-patterns, debugging-patterns, planning-patterns, github-research. It does NOT cover: architecture-patterns, frontend-patterns, brainstorming, session-memory (for some agents).

### 2.1 component-builder → builder — OK / ENHANCED

**Preserved:**
- TDD cycle (RED → GREEN → REFACTOR) - identical
- Memory First section - identical (path updated)
- SKILL_HINTS handling - identical
- Plan File Check gate - identical
- Pre-Implementation Checklist (API, UI, DB, All) - identical
- Memory Updates (Read-Edit-Verify) - identical
- TDD Evidence output section - identical
- Dev Journal format - identical
- Task Completion / TODO handling - identical
- Router Contract format - ENHANCED (added SPEC_COMPLIANCE, TIMESTAMP, AGENT_ID, FILES_MODIFIED, DEVIATIONS_FROM_PLAN)

**Enhanced for Agent Teams:**
- Added Pair Build Communication (SendMessage to live-reviewer after each module) - NEW
- Added "You OWN all file writes" rule - NEW
- Added Router Handoff section (pre-contract stable extraction) - NEW
- Process expanded from 6 to 10 steps (review request, wait for feedback, repeat, complete) - ENHANCED

**Missing frontmatter skills (relying on SKILL_HINTS):**
- `test-driven-development` — covered by SKILL_HINTS ✓
- `code-generation` — covered by SKILL_HINTS ✓
- `frontend-patterns` — NOT COVERED (skill doesn't exist in CC100x) ✗
- `architecture-patterns` — NOT COVERED (skill doesn't exist in CC100x) ✗

### 2.2 code-reviewer → security/performance/quality reviewers — ENHANCED

**Split is well-executed.** The single CC10x code-reviewer that checked security+quality+performance is now 3 specialists with deeper checklists.

**security-reviewer preserves + enhances:**
- OWASP Top 10 full checklist (NEW - CC10x just had "Security" as one line) - ENHANCED
- Auth Flow Verification 5-point checklist (NEW) - ENHANCED
- Quick scan commands (secrets, injection, dangerous patterns, CORS) - ENHANCED
- Confidence scoring (identical to CC10x) - OK
- Git Context section - OK
- Memory Notes / Router Contract - OK
- Added "Challenging Other Reviewers" section - NEW for Review Arena

**performance-reviewer preserves + enhances:**
- Database & Queries checklist (N+1, indexes, unbounded, over-fetch, pooling) - ENHANCED
- Memory & Resources checklist - NEW
- Frontend Performance checklist - NEW (partially absorbs frontend-patterns)
- API & Network checklist - NEW
- Quick scan commands - ENHANCED

**quality-reviewer preserves + enhances:**
- Code Patterns & Structure (naming, SRP, complexity, duplication, dead code) - ENHANCED
- Error Handling Quality table - NEW
- Test Coverage checks - NEW
- Architecture Adherence checks - NEW (partially absorbs architecture-patterns)
- Quick scan commands - ENHANCED

**FINDING [MINOR]:** CC10x code-reviewer had `cc10x:code-review-patterns` in frontmatter. CC100x reviewers do NOT have it in frontmatter — relying on SKILL_HINTS. If lead forgets to pass SKILL_HINTS, reviewers miss it.

### 2.3 silent-failure-hunter → hunter — OK / ENHANCED

**Preserved:**
- Core mission (zero tolerance for silent failures) - identical
- Red Flags table - ENHANCED (added `?.` chains, retry without notification)
- Severity Rubric (CRITICAL/HIGH/MEDIUM/LOW) - identical
- Classification Decision Tree - identical
- Process steps - identical
- CRITICAL Issues blocking rule - identical
- Memory Notes / Router Contract - identical logic
- Task Completion / TODO handling - identical

**Enhanced:**
- Added Scan Commands section (empty catches, log-only, generic messages, swallowed promises) - NEW
- Router Contract adds SPEC_COMPLIANCE, TIMESTAMP, AGENT_ID fields - ENHANCED

**Missing frontmatter skills:**
- `code-review-patterns` — covered by SKILL_HINTS (BUILD/DEBUG post-build) ✓
- `frontend-patterns` — NOT COVERED ✗
- `architecture-patterns` — NOT COVERED ✗

### 2.4 integration-verifier → verifier — ENHANCED

**Preserved:**
- Core mission (E2E validation with exit code evidence) - identical
- Process steps (understand, run tests, check patterns, test edges) - OK
- Rollback Decision (Option A/B/C) - identical
- Memory Notes / Router Contract - identical logic
- Task Completion / TODO handling - identical

**Enhanced:**
- Added Verification Commands section (npm test, build, tsc, lint, integration, e2e) - NEW
- Added Goal-Backward Lens section (Truths, Artifacts, Wiring) with commands - ENHANCED (was only in CC10x logic analysis doc)
- Added Stub Detection section (TODO markers, empty returns, empty handlers) - NEW
- Added Wiring Check Commands (Component→API, API→Database, Export/Import) - NEW
- Router Contract adds SPEC_COMPLIANCE, TIMESTAMP, AGENT_ID fields - ENHANCED

**Missing frontmatter skills:**
- `architecture-patterns` — NOT COVERED ✗
- `debugging-patterns` — NOT in frontmatter OR SKILL_HINTS for verifier ✗
- `frontend-patterns` — NOT COVERED ✗

**FINDING [MAJOR]: Verifier lost `debugging-patterns` skill**
CC10x integration-verifier had `cc10x:debugging-patterns` in frontmatter. CC100x verifier has neither frontmatter nor SKILL_HINTS coverage for debugging-patterns. The SKILL_HINTS table only passes debugging-patterns to investigator (DEBUG workflow), not verifier.

### 2.5 bug-investigator → investigator — MAJOR ARCHITECTURAL CHANGE + OK

**INTENTIONAL CHANGE:** CC10x bug-investigator was a WRITE agent that both investigated AND fixed bugs. CC100x investigator is READ-ONLY — it gathers evidence and the builder implements fixes. This is by design for Bug Court (competing hypotheses).

**Preserved:**
- Evidence-first core philosophy - identical
- Anti-Hardcode Gate (variant dimensions) - CARRIED OVER perfectly
- Context Retrieval 3-cycle pattern (DISPATCH→EVALUATE→REFINE) - CARRIED OVER
- Git History commands - identical
- Debug Attempt Format ([DEBUG-N]:) - CARRIED OVER
- Memory Notes section - OK

**Enhanced for Bug Court:**
- Added "Challenging Other Investigators" section - NEW
- Added Reproduction Evidence section (was implicit in CC10x) - ENHANCED
- Router Contract STATUS changed from FIXED|INVESTIGATING|BLOCKED to EVIDENCE_FOUND|INVESTIGATING|BLOCKED - correct for new role
- Added AGENT_ID: "investigator-{N}" for multiple investigators - ENHANCED

**FINDING [MAJOR]: Conditional Research lost explicit github-research skill reference**
CC10x bug-investigator: `Skill(skill="cc10x:github-research")` — explicit skill invocation
CC100x investigator: "web research via WebFetch" / "external research" — generic, no skill reference
The CC100x lead SKILL_HINTS table DOES list github-research for investigator, but the agent's own prompt doesn't tell it to invoke the skill. If SKILL_HINTS are passed, the agent loads it. But the agent's own Conditional Research section doesn't reference it.

**FINDING [MINOR]: Investigator lost TDD content**
CC10x had TDD Evidence section (RED/GREEN phases) since it implemented fixes. CC100x correctly removes this (now read-only). But the TDD knowledge from the `test-driven-development` skill is gone — the investigator can't recommend TDD-aware regression tests without it.

### 2.6 planner → planner — OK with losses

**Preserved:**
- Core mission (create plans, save to docs/plans/) - identical
- Memory First section - identical (path updated)
- Clarification Gate - identical
- Context Retrieval 3-cycle pattern - CARRIED OVER
- Process steps - identical
- Plan Format - ENHANCED (CC100x adds explicit template)
- Two-Step Save - identical
- Confidence Score - identical
- Memory Updates (Read-Edit-Verify) - identical
- Recommended Skills for BUILD section - identical
- Dev Journal / Router Contract - OK

**FINDING [MAJOR]: Planner lost brainstorming skill trigger**
CC10x planner: "If 3+ questions needed → `Skill(skill='cc10x:brainstorming')` for structured discovery"
CC100x planner: "If 3+ questions needed → consider structured discovery to gather all requirements at once"
The explicit skill reference is gone. The brainstorming skill doesn't exist in CC100x at all.

**FINDING [MAJOR]: Planner lost explicit github-research skill reference**
CC10x: `Skill(skill="cc10x:github-research")` for conditional research
CC100x: "web research via WebFetch" / "search GitHub for reference implementations" — generic

**Missing frontmatter skills:**
- `planning-patterns` — covered by SKILL_HINTS ✓
- `architecture-patterns` — NOT COVERED ✗
- `brainstorming` — NOT COVERED (doesn't exist) ✗
- `frontend-patterns` — NOT COVERED ✗

### 2.7 live-reviewer — NEW (no CC10x equivalent)

New agent for Pair Build workflow. Well-designed with:
- LGTM/STOP/NOTE response types
- Clear "When to STOP" criteria
- Quick review checklist
- Confidence scoring
- Communication protocol via SendMessage

---

## Phase 3: Skills

### 3.1 Matched Skills (CC10x → CC100x)

| CC10x Skill | CC100x Skill | Verdict | Notes |
|-------------|-------------|---------|-------|
| code-review-patterns | code-review-patterns | **IDENTICAL** | Name reference change only (router→lead) |
| debugging-patterns | debugging-patterns | **IDENTICAL** | Name reference change only |
| test-driven-development | test-driven-development | **IDENTICAL** | Name reference change only |
| code-generation | code-generation | **IDENTICAL** | Name reference change only |
| verification-before-completion | verification | **IDENTICAL** | Renamed, content identical |
| planning-patterns | planning-patterns | **MINOR DIFFS** | Execution Handoff rewritten for Agent Teams; emoji→text |
| session-memory | session-memory | **MINOR DIFFS** | Agent names updated; Concurrency Rule rewritten for Agent Teams (parallel reviewers/investigators) |
| github-research | github-research | **MINOR DIFFS** | Context7 MCP tools removed; fallback chain simplified; agent names updated |

**All 8 matched skills are properly transitioned.** Differences are appropriate adaptations for Agent Teams.

### 3.2 CC100x-Only Skills (New)

| CC100x Skill | Purpose | Quality |
|-------------|---------|---------|
| review-arena | Multi-perspective adversarial review (3 reviewers + challenge round) | **Comprehensive** — Two-stage review, signal quality, deduplication, conflict resolution, timeout handling |
| bug-court | Competing hypothesis debugging (N investigators + debate + builder fix) | **Comprehensive** — 7 phases, LSP tracing, cognitive biases, anti-hardcode gate, debug attempt tracking |
| pair-build | Real-time TDD pair programming (builder + live-reviewer + hunter + verifier) | **Comprehensive** — Plan-first gate, message-based sync, remediation loop, TDD verification checklist |
| router-contract | Universal YAML contract format for all agents | **Comprehensive** — STATUS values per agent, validation logic, circuit breaker, abort behavior, remediation loop |

**All 4 new skills are well-designed and appropriate for Agent Teams architecture.**

### 3.3 FINDING [MINOR]: github-research lost Context7 MCP
CC10x had Context7 MCP (resolve-library-id, query-docs) as a research tier. CC100x removed it entirely. May be intentional (tool not available) but reduces research capability.

---

## Phase 4: Missing Skills Investigation — CONFIRMED CRITICAL

### 4.1 FINDING [CRITICAL]: `architecture-patterns` — COMPLETELY MISSING (361 lines)

**CC10x usage:** Loaded by 5 of 6 agents (component-builder, code-reviewer, silent-failure-hunter, integration-verifier, planner)

**Content lost (not absorbed into any CC100x skill):**
- **Iron Law:** "NO ARCHITECTURE DESIGN BEFORE FUNCTIONALITY FLOWS ARE MAPPED"
- **Universal Questions** (7 questions to answer before designing)
- **Functionality-First Design Process** (3-phase: Map Flows → Map to Architecture → Design Components)
- **Architecture Views** (C4 Levels 1-3: System Context, Container, Component)
- **LSP-Powered Architecture Analysis** (dependency mapping, impact analysis, interface verification)
- **API Design** (functionality-aligned endpoint design, API Design Checklist)
- **Integration Patterns** (Retry, Circuit Breaker, Queue, WebSocket, Event Sourcing)
- **Observability Design** (Logging, Metrics, Alerts, Tracing)
- **Decision Framework** (structured ADR-like template with trade-offs)
- **Rationalization Prevention** (5 excuses blocked)
- **Output Format** (Architecture Design template)
- **Final Check** (6-point verification)

**Partial overlap:** CC100x quality-reviewer has "Architecture Adherence" as ONE checklist item. This is 1% of what architecture-patterns contained.

### 4.2 FINDING [CRITICAL]: `frontend-patterns` — COMPLETELY MISSING (583 lines!)

**CC10x usage:** Loaded by ALL 6 agents (every single CC10x agent had this skill)

**Content lost (not absorbed into any CC100x skill):**
- **Iron Law:** "NO UI DESIGN BEFORE USER FLOW IS UNDERSTOOD"
- **Design Thinking** (Purpose, Tone, Constraints, Differentiation)
- **Loading State Order** (Error → Loading → Empty → Data decision tree) — CRITICAL
- **Skeleton vs Spinner** decision matrix
- **Motion & Animation** rules (prefers-reduced-motion, compositor-friendly)
- **Error Handling Hierarchy** (inline → toast → banner → full screen)
- **Typography Rules** (ellipsis, quotes, units, shortcuts, tabular-nums, text-wrap)
- **Content Overflow Handling** (truncate, line-clamp, break-words, min-w-0)
- **Form Best Practices** (autocomplete, input types, inputMode, never block paste, spellcheck, unsaved changes, error focus)
- **Visual Design Checklist** (hierarchy, spacing, alignment, interactive states)
- **Visual Creativity** (anti-AI-slop rules: distinctive fonts, cohesive palette, no emoji icons, cursor-pointer)
- **Spatial Composition** (asymmetry, overlap, diagonal flow, grid-breaking)
- **Component Patterns** (Buttons, Forms with validation, Loading States, Error Messages — with actual code examples)
- **Responsive Design Checklist** (mobile/tablet/desktop breakpoints)
- **Performance Rules** (virtualize lists, no layout reads in render, lazy load, fetchpriority, preconnect, preload fonts)
- **URL & State Management** (URL-state sync patterns)
- **Touch & Mobile** (44px targets, no double-tap delay, modal scroll lock, safe areas)
- **Light/Dark Mode** (color-scheme, theme-color meta, contrast rules)
- **UX Review Checklist** (task completion, discoverability, feedback, error handling, efficiency)
- **Accessibility Review (WCAG 2.1 AA)** (keyboard, focus, labels, alt text, contrast, color-alone, screen reader)
- **Anti-patterns Blocklist** (11 specific anti-patterns with fixes: user-scalable=no, transition:all, outline-none, div-onClick, etc.)
- **UI States Checklist** (States + Buttons/Mutations + Data Handling)

**Partial overlap:** CC100x performance-reviewer has "Frontend Performance" as a 4-item checklist. CC100x code-review-patterns has "UX Review Checklist" and "Accessibility Review Checklist" (these were in the skill before CC100x, so they're preserved in the skill but reviewers may not LOAD the skill). This covers maybe 10% of what frontend-patterns contained.

### 4.3 FINDING [CRITICAL]: `brainstorming` — COMPLETELY MISSING (365 lines)

**CC10x usage:** Loaded by planner. Triggered by planner's "If 3+ questions needed" logic.

**Content lost:**
- **Iron Law:** "NO DESIGN WITHOUT UNDERSTANDING PURPOSE AND CONSTRAINTS"
- **One Question at a Time** methodology (sequential, not batch)
- **Multiple Choice Preferred** (AskUserQuestion with options)
- **Phase 1: Understand Context** (check project state before asking)
- **Phase 2: Explore the Idea** (5 sequential questions: Purpose, Users, Success Criteria, Constraints, Scope)
- **Phase 3: Explore Approaches** (2-3 options with trade-offs, always present alternatives)
- **Phase 4: Present Design Incrementally** (200-300 word sections with validation checkpoints)
- **Spec File Workflow** (read existing spec, expand, write back)
- **YAGNI Enforcement** (explicit ruthless scoping)
- **UI Mockup** (ASCII mockups for UI features)
- **Saving the Design** (Two saves: design file + memory update)
- **Rationalization Prevention** (5 excuses blocked)
- **Final Check** (8-point checklist)

**Impact:** CC100x planner's Clarification Gate says "consider structured discovery" but has no methodology for it. The brainstorming skill WAS that methodology.

---

## Phase 5: Cross-cutting Concerns

### 5.1 FINDING [CRITICAL]: SKILL_HINTS Table Is Incomplete

The CC100x lead's SKILL_HINTS table (the mechanism that bridges frontmatter skill stripping):

| Workflow | Skills Passed | MISSING vs CC10x |
|----------|--------------|-----------------|
| BUILD | TDD, code-gen | architecture-patterns, frontend-patterns, session-memory (for investigator in post-fix) |
| REVIEW | code-review-patterns | architecture-patterns, frontend-patterns |
| DEBUG | debugging-patterns | architecture-patterns, frontend-patterns, TDD (investigators could recommend TDD-aware tests) |
| PLAN | planning-patterns | architecture-patterns, frontend-patterns, brainstorming |

**Even if the 3 missing skills were created, they're not in the SKILL_HINTS table and wouldn't be passed to agents.**

### 5.2 FINDING [MAJOR]: Verification Skill Not in SKILL_HINTS

`cc100x:verification` exists and is in every agent's frontmatter. But it is NOT in the SKILL_HINTS table. This means the lead never explicitly mentions it when spawning teammates. The skill IS loaded via frontmatter, so this is a documentation inconsistency rather than a functional gap. However, it breaks the pattern of "SKILL_HINTS documents all skills agents use."

### 5.3 FINDING [MAJOR]: CC100x Bible Insufficient as Architecture Reference

CC10x Bible: 605 lines with Glossary, Task Types, Agent Output Requirements, Non-Optional Behaviors, Task State Transitions, Skill Loading Hierarchy.
CC100x Bible: 114 lines, mostly overview.

Most of the missing content IS covered in cc100x-lead SKILL.md (924 lines). But the bible should serve as a standalone architecture reference for maintainers. Currently it doesn't.

### 5.4 FINDING [MINOR]: Orchestration Logic Analysis Not Ported

CC10x had a dedicated 699-line "orchestration logic analysis" doc with:
- English Tricks catalog (permission-free operations, gate enforcement, rationalization prevention)
- Iron Laws catalog (comprehensive list across all skills)
- Agent-Specific Gates detail
- Hydration Pattern, 3-Task Rule, Context Retrieval Pattern
- Plan → Build Handoff pattern

These are engineering insights about HOW the orchestration works. Some are embedded in the lead SKILL.md (e.g., Context Retrieval is in planner and investigator). But as a cohesive reference document, it doesn't exist in CC100x.

### 5.5 OK: Agent Teams Adaptation Quality

The Agent Teams-specific adaptations are well-executed:
- Delegate mode enforcement ✓
- Teammate messaging (SendMessage) properly used ✓
- Task-based coordination with TaskCreate/TaskUpdate ✓
- READ-ONLY vs WRITE agent separation ✓
- Team shutdown protocol ✓
- Self-claim mode ✓
- Session interruption recovery ✓
- Model selection guidance ✓

### 5.6 OK: Router Contract Universal Adoption

All 9 CC100x agents output Router Contracts with proper STATUS values. The router-contract skill provides clear validation rules. This is well-designed.

### 5.7 OK: Memory Protocol Consistency

Memory First, Read-Edit-Verify, stable anchors, and template validation are consistent across all WRITE agents (builder, planner) and the lead. READ-ONLY agents properly defer memory to Memory Notes.

---

## Summary: All Findings by Severity

> **Fix Status**: ALL 11 findings FIXED on 2026-02-06. P0+P1 via `FIX-PLAN.md`. P2 in follow-up commit.

### CRITICAL (3) — ALL FIXED

1. **`architecture-patterns` skill MISSING** — 361 lines of architecture design methodology, loaded by 5/6 CC10x agents, zero representation in CC100x
   **FIXED**: Created `plugins/cc100x/skills/architecture-patterns/SKILL.md` (360 lines). Ported from CC10x with cc10x→cc100x name changes. Zero cc10x occurrences. All sections preserved: Iron Law, Universal Questions, Phase 1-3, Architecture Views (C4), LSP Analysis, API Design, Integration Patterns, Observability, Decision Framework, Rationalization Prevention.

2. **`frontend-patterns` skill MISSING** — 583 lines (largest CC10x skill), loaded by ALL 6 CC10x agents, zero representation in CC100x
   **FIXED**: Created `plugins/cc100x/skills/frontend-patterns/SKILL.md` (582 lines). Ported from CC10x with cc10x→cc100x name changes. Zero cc10x occurrences. All sections preserved: Iron Law, Design Thinking, Loading States, Accessibility (WCAG 2.1 AA), Forms, Animation, Component Patterns, Responsive Design, Performance Rules, Anti-patterns Blocklist (11 items), UI States Checklist.

3. **`brainstorming` skill MISSING** — 365 lines of structured discovery, loaded by planner, planner's "structured discovery" reference is now a dead link
   **FIXED**: Created `plugins/cc100x/skills/brainstorming/SKILL.md` (364 lines). Ported from CC10x with cc10x→cc100x name changes + memory path updates (.claude/cc10x/→.claude/cc100x/) + skill reference updates (cc10x:→cc100x:). Zero cc10x occurrences. All sections preserved: Iron Law, Phase 1-4, One Question at a Time, YAGNI, Spec File Workflow, Two Saves, UI Mockup, Rationalization Prevention.

### MAJOR (4) — ALL FIXED

4. **SKILL_HINTS table incomplete** — Even if missing skills were created, they're not wired into the distribution mechanism
   **FIXED**: Updated SKILL_HINTS table in `plugins/cc100x/skills/cc100x-lead/SKILL.md`. All 6 existing workflow rows now include `cc100x:architecture-patterns` + `cc100x:frontend-patterns`. PLAN row now includes `cc100x:brainstorming`. New verifier row added with `cc100x:debugging-patterns` + `cc100x:architecture-patterns` + `cc100x:frontend-patterns`. Table now has 9 rows (was 8).

5. **Verifier lost `debugging-patterns`** — CC10x verifier had it in frontmatter, CC100x verifier has neither frontmatter nor SKILL_HINTS for it
   **FIXED**: New verifier row in SKILL_HINTS table passes `cc100x:debugging-patterns` to verifier during BUILD/DEBUG post-verification phase.

6. **Planner/Investigator lost explicit `github-research` invocation** — Agent prompts say "web research via WebFetch" instead of referencing the skill
   **FIXED**: Updated `plugins/cc100x/agents/planner.md` Conditional Research section — now references `Skill(skill="cc100x:github-research")` with WebFetch fallback. Updated `plugins/cc100x/agents/investigator.md` Conditional Research section — same pattern.

7. **CC100x Bible is thin** — 114 lines vs 605 lines, missing Glossary, Task Types, Non-Optional Behaviors, etc.
   **FIXED**: Expanded `docs/cc100x-bible.md` from 114 to 380 lines. Added 8 sections: Glossary (CC100x Terms), Skills vs Agents (with Agent Teams differences), Orchestration Invariants (10 rules), Agent Chain Protocols (all 4 workflows with diagrams), Task Types and Prefixes, Non-Optional Behaviors (7 hard rules), Skill Loading Hierarchy (frontmatter table + SKILL_HINTS table for all 9 agents), Task State Transitions. Fixed Architecture Components table paths (agents/*.md, workflow skill paths).

### MINOR (4) — ALL FIXED

8. **github-research lost Context7 MCP** — Reduced research capability (may be intentional)
   **FIXED**: Restored Context7 MCP tools (`mcp__context7__resolve-library-id`, `mcp__context7__query-docs`) to `plugins/cc100x/skills/github-research/SKILL.md`. Added to allowed-tools, overview, availability check fallback chain, new Tier 1.5 section, and updated tier progression diagram. Skill now has full 4-tier fallback: Octocode+BrightData → Context7 → WebSearch/WebFetch → Ask User.

9. **Orchestration Logic Analysis not ported** — Engineering insights doc doesn't exist
   **FIXED**: Created `docs/cc100x-orchestration-logic-analysis.md` (828 lines, expanded from CC10x's 699). All sections preserved: English Tricks, Gate Enforcement, Iron Laws (15), Agent-Specific Gates, Context Retrieval Pattern, Hydration Pattern, 3-Task Rule, Goal-Backward Lens, Verification Flow, Memory Flow, Research Flow, Handoff Patterns. Updated for 9 agents, 16 skills, Agent Teams architecture (delegate mode, peer messaging, team coordination).

10. **Verification skill not in SKILL_HINTS table** — Documentation inconsistency (functional: verification is in every agent's frontmatter, so it loads without SKILL_HINTS)
    **FIXED**: Added clarifying note in cc100x-lead SKILL.md explaining that `router-contract`, `verification`, and `session-memory` load via frontmatter (unconditional) and are intentionally NOT in the SKILL_HINTS table.

11. **Investigator lost TDD-aware test recommendation capability** — Now READ-ONLY, no TDD skill (architectural decision: investigator gathers evidence, builder implements fix with TDD)
    **FIXED**: Added step 11 to investigator process: "Recommend regression test — Describe a failing test the builder should write FIRST (TDD: RED before GREEN)." Updated Recommended Fix output section to reference TDD approach. Investigator stays READ-ONLY but now recommends TDD-structured tests for the builder to implement.

---

## Recommended Actions (Priority Order)

### P0 — Must Fix (Critical) — ALL DONE
1. ~~**Create `cc100x:architecture-patterns` skill**~~ — DONE (360 lines)
2. ~~**Create `cc100x:frontend-patterns` skill**~~ — DONE (582 lines)
3. ~~**Create `cc100x:brainstorming` skill**~~ — DONE (364 lines)
4. ~~**Update SKILL_HINTS table in cc100x-lead**~~ — DONE (9 rows, all workflows covered)

### P1 — Should Fix (Major) — ALL DONE
5. ~~**Add `debugging-patterns` to verifier**~~ — DONE (new SKILL_HINTS row for verifier)
6. ~~**Add explicit `github-research` references**~~ — DONE (planner + investigator prompts updated)
7. ~~**Expand CC100x Bible**~~ — DONE (380 lines, 8 new sections)

### P2 — Nice to Have (Minor) — ALL DONE
8. ~~**Restore Context7 MCP to github-research**~~ — DONE (4-tier fallback restored)
9. ~~**Create CC100x Orchestration Logic Analysis doc**~~ — DONE (828 lines)
10. ~~**Add verification/session-memory/router-contract note to SKILL_HINTS**~~ — DONE (clarifying note)
11. ~~**Add TDD awareness to investigator**~~ — DONE (step 11 + output section updated)
