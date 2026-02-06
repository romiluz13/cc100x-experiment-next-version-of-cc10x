# CC100x Architecture Bible

> **Last Updated**: February 5, 2026
> **Status**: DESIGN PHASE
> **Foundation**: Built on Claude Code Agent Teams (experimental, Feb 5 2026)

---

## What CC100x Is

CC100x is a next-generation orchestration system built **from the ground up** on Claude Code Agent Teams. Instead of a router that spawns subagents one-by-one, CC100x uses a **team lead** that spawns **teammates** who communicate with each other directly.

### CC10x vs CC100x

| Dimension | CC10x (production) | CC100x (experimental) |
|-----------|--------------------|-----------------------|
| Communication | Router → Subagent → Router (one-way) | Teammates message each other (peer-to-peer) |
| Coordination | Router dispatches everything | Shared task list, self-claiming |
| Parallel work | Limited (reviewer ∥ hunter only) | Any teammates can work in parallel |
| Feedback | Post-implementation (review after build) | Real-time (review DURING build) |
| Lead role | Router (can see everything, mediates all) | Team lead in delegate mode (coordination only) |
| Debate | None (single reviewer perspective) | Adversarial (reviewers challenge each other) |
| Cost | Lower (subagent results summarized) | Higher (each teammate = separate Claude instance) |

### What Stays From CC10x

These are proven and still apply in CC100x:

- **Router Contract (YAML)**: Teammates output structured validation data. Lead validates.
- **Memory protocol**: `.claude/cc100x/` with activeContext/patterns/progress files.
- **Decision tree**: ERROR→DEBUG, PLAN→PLAN, REVIEW→REVIEW, DEFAULT→BUILD.
- **TDD enforcement**: Builder teammate follows RED→GREEN→REFACTOR.
- **Confidence gating**: Reviewers need >=80 confidence to report findings.
- **Task DAGs**: Agent Teams use shared task list with dependencies.

### What's New in CC100x

- **Peer-to-peer messaging**: Teammates talk directly to each other.
- **Adversarial review**: Multiple reviewers challenge each other's findings.
- **Delegate mode**: Lead is structurally prevented from implementing.
- **Self-claiming**: Teammates pick up unblocked tasks autonomously.
- **Real-time feedback**: Builder gets review feedback DURING implementation.
- **Competing hypotheses**: Multiple investigators debug in parallel.

---

## Workflows

### REVIEW: "Review Arena"

Spawn 3 specialized reviewer teammates who challenge each other:

```
Team Lead (delegate mode)
├── Security Reviewer (auth, injection, secrets, OWASP)
├── Performance Reviewer (N+1, loops, memory, bundle size)
└── Quality Reviewer (patterns, naming, error handling, duplication)
```

Flow: Each reviews independently → They challenge each other via messaging → Lead collects consensus → Outputs unified Router Contract.

### DEBUG: "Bug Court"

Spawn 3+ investigator teammates with competing hypotheses:

```
Team Lead (delegate mode)
├── Investigator A: Hypothesis 1
├── Investigator B: Hypothesis 2
└── Investigator C: Hypothesis 3
```

Flow: Each investigates their hypothesis → They debate and try to disprove each other → Surviving hypothesis = root cause → Winner implements fix → Review chain follows.

### BUILD: "Pair Build"

Builder + Live Reviewer as teammates:

```
Team Lead (delegate mode)
├── Builder (implements code, owns all file writes)
├── Live Reviewer (reads code as written, provides real-time feedback)
└── Hunter (joins after implementation complete)
```

Flow: Builder implements module by module → After each module, messages reviewer → Reviewer provides inline feedback → Builder fixes immediately → Final review + hunt → Verification.

### PLAN

Same as CC10x (single planner). Agent Teams add no value for solo planning.

---

## Architecture Components

| Component | CC100x Implementation |
|-----------|----------------------|
| **Lead Skill** | `skills/cc100x-lead/SKILL.md` - Team creation, delegation, validation |
| **Agents** | `agents/*.md` - Agent definitions for each teammate role (9 agents) |
| **Workflow Skills** | `skills/pair-build/`, `skills/review-arena/`, `skills/bug-court/` - Workflow definitions |
| **Domain Skills** | `skills/*/SKILL.md` - Expertise skills loaded via SKILL_HINTS (16 total) |
| **Memory** | `.claude/cc100x/` - Same 3-file system as CC10x |
| **Router Contract** | Same YAML format, output by all teammates. Defined in `skills/router-contract/` |

---

## Glossary (CC100x Terms)

- **Lead**: The execution engine defined by `plugins/cc100x/skills/cc100x-lead/SKILL.md`. Runs in delegate mode (coordination only, no code implementation).
- **Teammate**: A separate Claude Code instance spawned by the lead. Each has its own context window, can message other teammates directly, and CAN see CLAUDE.md + project skills.
- **Team Lead**: The main Claude Code session that creates the team, spawns teammates, assigns tasks, validates outputs, and persists memory.
- **Workflow**: One of BUILD (Pair Build), DEBUG (Bug Court), REVIEW (Review Arena), PLAN.
- **Agents**: 9 CC100x agent definitions in `plugins/cc100x/agents/`: builder, security-reviewer, performance-reviewer, quality-reviewer, live-reviewer, hunter, verifier, investigator, planner.
- **Skills**: 16 specialized instruction sets in `plugins/cc100x/skills/*/SKILL.md`.
- **Memory**: `.claude/cc100x/{activeContext.md, patterns.md, progress.md}`.
- **Router Contract**: Machine-readable YAML section in teammate output for validation.
- **Dev Journal**: User transparency section in teammate output (narrative of what was done).
- **Task List**: Shared work items at `~/.claude/tasks/{team-name}/`. Teammates claim and complete tasks. Dependencies managed automatically.
- **Mailbox**: Messaging system for inter-teammate communication. Messages delivered automatically.

---

## Skills vs Agents (Claude Code + Agent Teams Concepts)

> This section documents the Claude Code platform concepts that CC100x is built on.

### What is a Skill?

A **skill** is a Markdown file (`SKILL.md`) with optional YAML frontmatter. It provides instructions, reference material, or task workflows that teach Claude how to do something. Skills do NOT execute — they INSTRUCT.

**Skill frontmatter fields:**
```yaml
name: skill-name          # Identifier (lowercase, hyphens)
description: "..."        # When Claude should use this skill
allowed-tools: Read, Grep # Tools that skip permission prompts when skill is active
```

**Key facts:**
- `allowed-tools` is **NOT runtime enforcement**. It defines which tools skip permission prompts. The agent's `tools:` field controls actual tool availability.
- Skills are loaded as text context — injected into the agent's system prompt.
- Skills cannot call tools themselves. They instruct the hosting agent to call tools.

### What is an Agent (Teammate)?

An **agent** is a Markdown file with YAML frontmatter that defines an independent Claude Code session. When the lead spawns a teammate, Claude Code creates a new process with its own context window, tools, and instructions.

**Agent frontmatter fields:**
```yaml
name: agent-name          # Identifier
tools: Read, Edit, Bash   # Actual tool allowlist (enforced at runtime)
skills: skill-a, skill-b  # Skills to preload (full content injected at startup)
context: fork             # Run in isolated subprocess
model: inherit            # Model to use
```

**Key facts:**
- `tools:` is the **actual runtime allowlist**. Agent can ONLY use tools listed here.
- `skills:` preloads full skill content into the agent's system prompt at startup. Agent does NOT need to call `Skill()` for preloaded skills.
- `context: fork` means the agent runs in a fresh context.
- **Agent Teams difference:** Unlike CC10x subagents, teammates CAN see CLAUDE.md + project skills. Lead's conversation history does NOT carry over. SKILL_HINTS bridge the gap for conditional and user-global skills.

### Skills vs Agents — The Distinction

| Aspect | Skill | Agent (Teammate) |
|--------|-------|------------------|
| **Nature** | Instructions (text) | Execution unit (separate Claude instance) |
| **Runs as** | Context injected into an agent | Independent process with own context window |
| **Can use tools?** | No — instructs the hosting agent to use tools | Yes — has its own `tools:` allowlist |
| **Can see CLAUDE.md?** | If loaded into teammate context | Yes (Agent Teams teammates see project context) |
| **Can message peers?** | N/A | Yes — via SendMessage (peer-to-peer) |
| **Loaded via** | Agent frontmatter `skills:` (automatic) or `Skill()` call (on-demand) | Lead spawns via `Task(subagent_type="...", team_name="...")` |
| **Frontmatter tool field** | `allowed-tools` (permission hint, NOT enforcement) | `tools` (actual allowlist, enforced) |

### How CC100x Uses This Architecture

**9 Agents (teammate types):**
- `builder` — Builds features (has Edit, Write, Bash). WRITE agent.
- `investigator` — Gathers bug evidence (READ-ONLY in Bug Court; builder implements fix).
- `planner` — Creates plans (has Edit, Write, Bash for plan files + memory). WRITE agent.
- `security-reviewer` — Security review (READ-ONLY).
- `performance-reviewer` — Performance review (READ-ONLY).
- `quality-reviewer` — Quality review (READ-ONLY).
- `live-reviewer` — Real-time review during Pair Build (READ-ONLY).
- `hunter` — Finds silent failures (READ-ONLY).
- `verifier` — Verifies E2E (READ-ONLY).

**16 Skills (instruction sets loaded into agents):**
- `cc100x-lead` — Orchestration engine (loaded by main Claude, not agents)
- `session-memory` — Memory protocol (WRITE agents only: builder, planner)
- `router-contract` — Router Contract format (all agents)
- `verification` — Verification gates (all agents)
- `review-arena` — Review Arena workflow protocol
- `bug-court` — Bug Court workflow protocol
- `pair-build` — Pair Build workflow protocol
- `code-generation` — Code writing patterns (builder)
- `test-driven-development` — TDD protocol (builder)
- `debugging-patterns` — Debug methodology (investigator, verifier)
- `code-review-patterns` — Review methodology (reviewers, hunter)
- `planning-patterns` — Plan writing (planner)
- `brainstorming` — Idea exploration (planner)
- `architecture-patterns` — Architecture design (all agents via SKILL_HINTS)
- `frontend-patterns` — Frontend patterns (all agents via SKILL_HINTS)
- `github-research` — External research (conditional via SKILL_HINTS)

**Why this separation:**
1. **Skills are reusable** — `architecture-patterns` loads into all agents. One source of truth for architecture rules.
2. **Agents are isolated** — security-reviewer cannot accidentally edit files because its `tools:` field excludes Edit/Write. This is enforced by Claude Code, not by English instruction.
3. **Skills compose** — An agent's behavior is the combination of its base instructions + all preloaded skills + SKILL_HINTS skills.
4. **SKILL_HINTS bridge the gap** — The lead (running in main context) detects which skills are relevant per workflow and passes them via SKILL_HINTS in the teammate's prompt.

### External Skill Conflict Risk (Design Decision)

Claude Code auto-loads descriptions of ALL installed skills (global + project + plugins) into the main context. External skills with broad trigger descriptions can conflict with CC100x routing — Claude might invoke them instead of or alongside the lead. **CLAUDE.md's Complementary Skills table is the ONLY approved channel for external skills.** It ensures the user has explicitly vetted compatibility. Do not implement auto-discovery of installed skills.

---

## Orchestration Invariants (Never Break)

1. **Lead is the ONLY entry point.** Every development task must pass through it.
2. **Lead operates in delegate mode.** Coordination only, no code implementation.
3. **Memory load is mandatory before any decision.**
4. **Task-based orchestration is mandatory.** All workflows use tasks with dependencies.
5. **Workflow selection uses the decision tree in priority order.**
6. **Teammate chain must complete.** No workflow is done until its chain is complete.
7. **No two teammates edit the same file.** File ownership prevents conflicts.
8. **Teammates send Router Contracts; lead validates and persists.**
9. **Research is prerequisite if triggered, and MUST be persisted.**
10. **Memory must be updated at the end of a workflow.**

---

## Agent Chain Protocols (CC100x Workflows)

### BUILD Chain: "Pair Build"
```
builder ──┬── live-reviewer (concurrent, real-time feedback)
          │
          ▼
       hunter
          │
          ▼
       verifier
          │
          ▼
    Memory Update
```
**Pair rule:** builder + live-reviewer work concurrently. Builder messages live-reviewer after each module. Live-reviewer responds LGTM/STOP/NOTE.
**Do not run hunter until builder completes.**
**Do not run verifier until hunter completes.**

### DEBUG Chain: "Bug Court"
```
investigators (2-5 parallel) → debate round → builder fix → quality-reviewer → hunter → verifier → Memory Update
```
**Parallel rule:** All investigators run simultaneously, each championing one hypothesis.
**Debate phase:** Investigators challenge each other via peer messaging.
**Fix phase:** Winning hypothesis → builder implements fix using TDD.
**Post-fix:** Abbreviated Review Arena (quality-reviewer only) + hunter + verifier.

### REVIEW Chain: "Review Arena"
```
security-reviewer ──┐
performance-reviewer ├── challenge round → consensus → Memory Update
quality-reviewer ───┘
```
**Parallel rule:** All 3 reviewers run simultaneously, independently.
**Challenge phase:** Lead shares findings, reviewers debate via peer messaging.
**Consensus:** Lead collects verdicts, resolves conflicts (majority rules, security wins for CRITICAL).

### PLAN Chain
```
planner → Memory Update
```
Same as CC10x. Agent Teams add no value for solo planning.

---

## Task Types and Prefixes

| Type | Subject Prefix | Created By | Purpose | Auto-Execute? |
|------|---------------|------------|---------|---------------|
| Workflow | `CC100X BUILD:` / `DEBUG:` / etc. | Lead | Parent workflow task | N/A |
| Agent | `CC100X {agent}:` | Lead | Teammate work item | Yes |
| Evidence-only | `CC100X REM-EVIDENCE:` | Lead | Re-run commands for missing evidence | Yes |
| Code changes | `CC100X REM-FIX:` | Lead | Fix issues found by reviewer/hunter | Yes (triggers re-review) |
| Follow-up | `CC100X TODO:` | Teammates | Non-blocking discoveries for user action | **No** (user decides) |

---

## Non-Optional Behaviors (Hard Rules)

- **Never stop after one teammate.** Complete the workflow chain.
- **Never claim completion without verification evidence.**
- **No production code without failing test first (TDD).**
- **No architecture/plan/design before flows are mapped.**
- **Lead never implements code (delegate mode).**
- **No two teammates edit the same file.**
- **Teammates send Router Contracts; lead validates before proceeding.**

---

## Skill Loading Hierarchy (Definitive)

### Mechanism 1: Agent Frontmatter `skills:` (PRELOAD — Automatic)

Skills listed in agent frontmatter load automatically. No `Skill()` calls needed.

| Agent | Frontmatter Skills |
|-------|-------------------|
| builder | session-memory, router-contract, verification |
| security-reviewer | router-contract, verification |
| performance-reviewer | router-contract, verification |
| quality-reviewer | router-contract, verification |
| live-reviewer | router-contract, verification |
| hunter | router-contract, verification |
| verifier | router-contract, verification |
| investigator | router-contract, verification |
| planner | session-memory, router-contract, verification |

### Mechanism 2: Lead's SKILL_HINTS (Conditional — On Demand)

Lead detects workflow type and passes SKILL_HINTS per teammate. Agent invokes via `Skill(skill="{name}")`.

| Detected Pattern | Skill | Agents |
|------------------|-------|--------|
| **BUILD workflow** | TDD, code-generation, architecture-patterns, frontend-patterns | builder |
| **REVIEW workflow** | code-review-patterns, architecture-patterns, frontend-patterns | all 4 reviewers |
| **DEBUG workflow** | debugging-patterns, architecture-patterns, frontend-patterns | investigator |
| **PLAN workflow** | planning-patterns, brainstorming, architecture-patterns, frontend-patterns | planner |
| **Post-build** (hunter) | code-review-patterns, architecture-patterns, frontend-patterns | hunter |
| **Post-verification** (verifier) | debugging-patterns, architecture-patterns, frontend-patterns | verifier |
| External research trigger | github-research | planner, investigator |

**Workflow-based skills are ALWAYS passed for the matching workflow.** They are not conditional — every BUILD gets TDD+code-gen+architecture+frontend, every REVIEW gets code-review-patterns+architecture+frontend, etc.

**Also check CLAUDE.md Complementary Skills table and include matching skills in SKILL_HINTS.**

---

## Task State Transitions (Non-Negotiable)

```
┌─────────┐       ┌─────────────┐       ┌───────────┐
│ pending │──────>│ in_progress │──────>│ completed │
└─────────┘       └─────────────┘       └───────────┘
     │                   │
     │                   │
     └───────────────────┴──────────> deleted
```

**State Transitions:**
- `pending` → `in_progress`: When teammate starts work
- `in_progress` → `completed`: When teammate finishes
- Any → `deleted`: When task removed

**A task is available when:**
- status = `pending`
- blockedBy list is empty (all dependencies resolved)

**Blocked Tasks:**
- Task with non-empty `blockedBy` cannot become `in_progress`
- When blocking task completes, blocked task automatically becomes available

---

## Final Reminder

This Bible is the contract.
Any change to routing, task orchestration, parallel execution, or memory gates **must be evaluated against this doc first**.

---

## Reference Materials

All reference files are in `reference/`:
- `agent-teams-complete-docs.md` - Full Agent Teams documentation
- `cc10x-orchestration-bible.md` - CC10x architecture (what we evolved from)
- `cc10x-router-SKILL.md` - CC10x router logic
- `*.md` (agent files) - CC10x agent definitions
- `anthropic-2026-features-research.md` - All Anthropic 2026 features
