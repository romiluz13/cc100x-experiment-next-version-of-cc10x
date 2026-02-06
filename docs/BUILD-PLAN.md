# CC100x Build Plan (A-to-Z)

> **This is the master plan. Follow it phase by phase.**
> **Do not skip phases. Do not reorder. Each phase depends on the previous.**
> **Read reference files BEFORE writing anything.**

---

## Phase 0: Plugin Scaffold

**Goal**: Create the Claude Code plugin structure so CC100x can be installed.

### Read first:
- `reference/cc10x-plugin-structure/plugin.json` (CC10x plugin manifest - copy and modify)
- `reference/cc10x-plugin-structure/marketplace.json` (CC10x marketplace config - copy and modify)
- `reference/cc10x-plugin-structure/.gitignore` (CC10x gitignore - copy)

### Files to create:

#### 0.1 Plugin manifest
**File**: `plugins/cc100x/.claude-plugin/plugin.json`
```json
{
  "name": "cc100x",
  "version": "0.1.0",
  "description": "Next-generation orchestration built on Agent Teams. Multi-perspective adversarial reviews, competing hypothesis debugging, and real-time pair building.",
  "author": {
    "name": "Rom Iluz",
    "email": "rom@iluz.net",
    "url": "https://github.com/romiluz13"
  },
  "homepage": "https://github.com/romiluz13/cc100x",
  "repository": "https://github.com/romiluz13/cc100x",
  "license": "MIT",
  "keywords": [
    "agent-teams",
    "orchestration",
    "adversarial-review",
    "competing-hypotheses",
    "pair-building",
    "code-review",
    "tdd",
    "debugging",
    "claude-code"
  ]
}
```

#### 0.2 Move skill/teammate/protocol folders into plugin structure
Reorganize so the plugin root is `plugins/cc100x/`:
```
plugins/cc100x/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── cc100x-lead/
│       └── SKILL.md
├── agents/              # Note: "agents" not "teammates" (Claude Code convention)
│   ├── security-reviewer.md
│   ├── performance-reviewer.md
│   ├── quality-reviewer.md
│   ├── builder.md
│   ├── live-reviewer.md
│   ├── hunter.md
│   ├── verifier.md
│   ├── investigator.md
│   └── planner.md
└── skills/
    ├── cc100x-lead/SKILL.md
    ├── review-arena/SKILL.md        # Protocol as a skill
    ├── bug-court/SKILL.md           # Protocol as a skill
    ├── pair-build/SKILL.md          # Protocol as a skill
    ├── session-memory/SKILL.md      # Ported from cc10x
    ├── verification/SKILL.md        # Ported from cc10x
    └── router-contract/SKILL.md     # Contract format spec
```

#### 0.3 Git init
```bash
cd /Users/rom.iluz/Dev/cc10x_v5/cc100x
git init
```

**Validation**: `ls plugins/cc100x/.claude-plugin/plugin.json` returns the file.

---

## Phase 1: Core Lead Skill

**Goal**: Write the team lead that orchestrates everything.

### Read first:
- `reference/agent-teams-complete-docs.md` (how Agent Teams work)
- `reference/cc10x-skills/cc10x-router.md` (what we're evolving from)
- `reference/cc10x-orchestration-bible.md` (orchestration invariants)
- `docs/cc100x-bible.md` (CC100x architecture)

### File to create:
**File**: `plugins/cc100x/skills/cc100x-lead/SKILL.md`

**Content spec** (what must be in this file):

```yaml
---
name: cc100x-lead
description: |
  THE ONLY ENTRY POINT FOR CC100X. Orchestrates Agent Teams for development workflows.
  Triggers on: build, debug, review, plan, fix, create, implement, etc.
---
```

**The SKILL.md must include:**

1. **Decision Tree** (same as CC10x):
   - ERROR → DEBUG (Bug Court protocol)
   - PLAN → PLAN (single planner, same as CC10x)
   - REVIEW → REVIEW (Review Arena protocol)
   - DEFAULT → BUILD (Pair Build protocol)

2. **Memory Protocol** (same as CC10x but `.claude/cc100x/`):
   - Step 1: `mkdir -p .claude/cc100x`
   - Step 2: Read activeContext.md, patterns.md, progress.md
   - Template validation gate (auto-heal missing sections)

3. **Team Creation Logic**:
   For each workflow, define HOW to create the agent team:
   - What teammates to spawn (names, roles, models)
   - What spawn prompts to use (reference teammate template files)
   - When to use delegate mode (always for REVIEW and DEBUG, optional for BUILD)
   - How to set up the shared task list with dependencies

4. **Protocol Selection**:
   - REVIEW → Load `cc100x:review-arena` skill
   - DEBUG → Load `cc100x:bug-court` skill
   - BUILD → Load `cc100x:pair-build` skill
   - PLAN → Direct planner invocation (no team needed)

5. **Post-Team Validation**:
   - After team completes, lead collects Router Contracts from all teammates
   - Validates using same contract rules as CC10x
   - Handles conflicts (e.g., one reviewer approves, another finds critical issues)
   - Persists memory using Memory Notes from teammates

6. **Task List Management**:
   - Create tasks before spawning team
   - Teammates self-claim tasks
   - Lead monitors completion
   - Memory Update task at end (same as CC10x)

**IMPORTANT**: The lead skill should instruct Claude to use **natural language** to create the team. Example:
```
Create an agent team for code review. Use delegate mode.
Spawn 3 reviewer teammates:
- Security reviewer: [spawn prompt from security-reviewer.md template]
- Performance reviewer: [spawn prompt from performance-reviewer.md template]
- Quality reviewer: [spawn prompt from quality-reviewer.md template]
Have them review, then challenge each other's findings.
Wait for all teammates to complete before synthesizing.
```

---

## Phase 2: Router Contract Skill

**Goal**: Define the contract format used by ALL teammates.

### Read first:
- `reference/cc10x-orchestration-bible.md` (Router Contract section)
- Any CC10x agent file in `reference/cc10x-agents/` (see "Router Contract (MACHINE-READABLE)" section at the bottom)

### File to create:
**File**: `plugins/cc100x/skills/router-contract/SKILL.md`

**Content spec**: Define the YAML contract format that all teammates must output:

```yaml
STATUS: [PASS|FAIL|APPROVE|CHANGES_REQUESTED|CLEAN|ISSUES_FOUND|FIXED|PLAN_CREATED]
CONFIDENCE: [0-100]
CRITICAL_ISSUES: [count]
BLOCKING: [true|false]
REQUIRES_REMEDIATION: [true|false]
REMEDIATION_REASON: [null or description]
MEMORY_NOTES:
  learnings: ["..."]
  patterns: ["..."]
  verification: ["..."]
```

Include per-agent STATUS values (same as CC10x bible).

---

## Phase 3: Session Memory Skill

**Goal**: Port CC10x's memory protocol for CC100x.

### Read first:
- `reference/cc10x-orchestration-bible.md` (Memory Protocol section)
- `reference/cc10x-skills/session-memory.md` (CC10x session memory skill to port)

### File to create:
**File**: `plugins/cc100x/skills/session-memory/SKILL.md`

**Content spec**: Same as CC10x session-memory but:
- Path: `.claude/cc100x/` (not `.claude/cc10x/`)
- Same 3 files: activeContext.md, patterns.md, progress.md
- Same section headers and anchors
- Same Read-Edit-Verify pattern
- Same template validation gate

---

## Phase 4: Verification Skill

**Goal**: Port CC10x's verification-before-completion.

### Read first:
- `reference/cc10x-orchestration-bible.md` (Non-Optional Behaviors section)
- `reference/cc10x-skills/verification-before-completion.md` (CC10x verification skill to port)

### File to create:
**File**: `plugins/cc100x/skills/verification/SKILL.md`

**Content spec**: Evidence-based completion gates. No claims without exit codes. Same principles as CC10x.

---

## Phase 5: Review Arena (Protocol + Teammates)

**Goal**: Build the multi-perspective adversarial review system.

### Read first:
- `reference/agent-teams-complete-docs.md` (Use case: Parallel Code Review)
- `reference/cc10x-agents/code-reviewer.md` (CC10x code reviewer as reference)
- `reference/cc10x-agents/silent-failure-hunter.md` (CC10x hunter as reference)
- `reference/cc10x-skills/code-review-patterns.md` (CC10x review patterns skill)
- `docs/cc100x-bible.md` (Review Arena section)

### Files to create:

#### 5.1 Review Arena Protocol
**File**: `plugins/cc100x/skills/review-arena/SKILL.md`

**Content spec**:
```yaml
---
name: review-arena
description: "Multi-perspective adversarial code review using Agent Teams"
---
```

Must define:
1. **Team composition**: 3 reviewer teammates (security, performance, quality)
2. **Phase 1 - Independent Review**: Each reviewer scans code from their perspective
3. **Phase 2 - Challenge Round**: Reviewers message each other to challenge findings
   - "Security reviewer: I found X. Performance reviewer, does this affect your findings?"
   - "Performance reviewer: Your fix for X would introduce an N+1 query. Here's why..."
4. **Phase 3 - Consensus**: Lead collects unified findings
5. **Output format**: Unified Router Contract with all perspectives merged
6. **Conflict resolution**: What happens when reviewers disagree (e.g., security says CRITICAL, performance says acceptable)

#### 5.2 Security Reviewer Agent
**File**: `plugins/cc100x/agents/security-reviewer.md`

**Content spec**:
```yaml
---
name: security-reviewer
description: "Security-focused code reviewer for Review Arena"
model: inherit
color: red
context: fork
tools: Read, Bash, Grep, Glob, Skill, LSP, WebFetch
skills: cc100x:router-contract, cc100x:verification
---
```

Body must include:
- Security review checklist (auth, injection, secrets, OWASP top 10, XSS, CSRF)
- Confidence scoring (>=80 to report)
- Output format with Router Contract
- Memory Notes section (READ-ONLY agent)
- Instructions for challenging other reviewers' findings via messaging

#### 5.3 Performance Reviewer Agent
**File**: `plugins/cc100x/agents/performance-reviewer.md`

Same pattern as security-reviewer but focused on:
- N+1 queries, unnecessary loops, memory leaks
- Bundle size, lazy loading, caching
- Database query optimization
- API response times, pagination

#### 5.4 Quality Reviewer Agent
**File**: `plugins/cc100x/agents/quality-reviewer.md`

Same pattern but focused on:
- Code patterns, naming, complexity
- Error handling quality
- Test coverage gaps
- Duplication, dead code
- Architecture adherence

---

## Phase 6: Bug Court (Protocol + Teammates)

**Goal**: Build the competing hypothesis debugging system.

### Read first:
- `reference/agent-teams-complete-docs.md` (Use case: Competing Hypotheses)
- `reference/cc10x-agents/bug-investigator.md` (CC10x bug investigator as reference)
- `reference/cc10x-skills/debugging-patterns.md` (CC10x debugging patterns skill)
- `docs/cc100x-bible.md` (Bug Court section)

### Files to create:

#### 6.1 Bug Court Protocol
**File**: `plugins/cc100x/skills/bug-court/SKILL.md`

**Content spec**:
```yaml
---
name: bug-court
description: "Competing hypothesis debugging using Agent Teams"
---
```

Must define:
1. **Hypothesis generation**: Lead generates 3-5 hypotheses based on error/symptoms
2. **Team composition**: One investigator per hypothesis
3. **Phase 1 - Investigation**: Each investigator gathers evidence for their hypothesis
4. **Phase 2 - Debate**: Investigators try to disprove each other
   - "Investigator A: My evidence shows X caused the bug. Investigator B, can you disprove this?"
   - "Investigator B: Your theory doesn't explain why the bug only happens on Tuesdays. Here's my counter-evidence..."
5. **Phase 3 - Verdict**: Lead determines which hypothesis survived
6. **Phase 4 - Fix**: Winning investigator implements the fix (TDD: regression test first)
7. **Phase 5 - Review**: Review Arena runs on the fix

#### 6.2 Investigator Agent
**File**: `plugins/cc100x/agents/investigator.md`

**Content spec**:
```yaml
---
name: investigator
description: "Bug hypothesis investigator for Bug Court"
model: inherit
color: red
context: fork
tools: Read, Edit, Write, Bash, Grep, Glob, Skill, LSP, WebFetch
skills: cc100x:session-memory, cc100x:router-contract, cc100x:verification
---
```

Body must include:
- Evidence-first debugging (LOG FIRST)
- Hypothesis investigation methodology
- TDD for fix (RED → GREEN)
- Anti-hardcode gate (variant coverage)
- Instructions for debating other investigators
- Router Contract output

---

## Phase 7: Pair Build (Protocol + Teammates)

**Goal**: Build the real-time pair programming system.

### Read first:
- `reference/agent-teams-complete-docs.md` (Best practices: file conflicts, context)
- `reference/cc10x-agents/component-builder.md` (CC10x builder as reference)
- `reference/cc10x-agents/code-reviewer.md` (CC10x reviewer as reference)
- `reference/cc10x-skills/test-driven-development.md` (CC10x TDD skill to port)
- `docs/cc100x-bible.md` (Pair Build section)

### Files to create:

#### 7.1 Pair Build Protocol
**File**: `plugins/cc100x/skills/pair-build/SKILL.md`

**Content spec**:
Must define:
1. **Team composition**: Builder + Live Reviewer + Hunter (joins later)
2. **File ownership**: Builder owns ALL file writes. Reviewer is READ-ONLY. No conflicts.
3. **Implementation loop**:
   - Builder implements one module/component
   - Builder messages reviewer: "Review src/auth/middleware.ts"
   - Reviewer reads, messages back feedback
   - Builder fixes inline, continues to next module
4. **Completion signal**: Builder messages "Implementation complete"
5. **Final scan**: Hunter joins for silent failure audit
6. **Verification**: Verifier runs E2E tests
7. **TDD enforcement**: Builder still follows RED → GREEN → REFACTOR

#### 7.2 Builder Agent
**File**: `plugins/cc100x/agents/builder.md`

Port from CC10x component-builder but add:
- Instructions for messaging live-reviewer after each module
- How to handle reviewer feedback (fix inline, then continue)
- Completion signal protocol

#### 7.3 Live Reviewer Agent
**File**: `plugins/cc100x/agents/live-reviewer.md`

New agent. READ-ONLY. Reviews code as builder writes it:
- Responds to builder's review requests via messaging
- Provides inline feedback (not a full review, just critical issues)
- Focuses on: security, correctness, patterns
- Can say "LGTM" to let builder continue or "STOP: [issue]" to flag a problem

#### 7.4 Hunter Agent
**File**: `plugins/cc100x/agents/hunter.md`

Port from `reference/cc10x-agents/silent-failure-hunter.md`. Same role, same output format.

#### 7.5 Verifier Agent
**File**: `plugins/cc100x/agents/verifier.md`

Port from `reference/cc10x-agents/integration-verifier.md`. Same role, same output format.

---

## Phase 8: Planner Agent

**Goal**: Port the planner for PLAN workflow (no Agent Teams needed).

### Read first:
- `reference/cc10x-agents/planner.md` (CC10x planner)
- `reference/cc10x-skills/planning-patterns.md` (CC10x planning skill)

### File to create:
**File**: `plugins/cc100x/agents/planner.md`

Port from CC10x planner. Adjust memory paths to `.claude/cc100x/`. Adjust skill references to `cc100x:*`.

---

## Phase 9: CLAUDE.md for Users

**Goal**: Create the CLAUDE.md that activates CC100x for any project.

### File to create:
**File**: `CLAUDE.md` (at repo root, for users to copy)

**Content spec**:
```markdown
# CC100x Orchestration (Agent Teams)

IMPORTANT: ALWAYS invoke cc100x-lead on ANY development task.
IMPORTANT: Requires Agent Teams enabled: CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

[CC100x]|entry: ./plugins/cc100x/skills/cc100x-lead/SKILL.md
```

---

## Phase 10: README and Publishing

**Goal**: Write README, create GitHub repo, publish to marketplace.

### Read first:
- `reference/cc10x-plugin-structure/README.md` (CC10x README - reference for cc100x)
- `reference/cc10x-plugin-structure/CLAUDE.md` (CC10x user CLAUDE.md - reference)
- `reference/cc10x-plugin-structure/marketplace.json` (CC10x marketplace config)
- `reference/cc10x-plugin-structure/LICENSE` (MIT license - copy)

### Files to create:

#### 10.1 README.md
Standard GitHub README with:
- What CC100x is (one paragraph)
- How it works (diagram of Agent Teams architecture)
- Installation instructions:
  ```bash
  # 1. Enable Agent Teams
  # Add to ~/.claude/settings.json:
  # "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" }

  # 2. Add marketplace
  /plugin marketplace add romiluz13/cc100x

  # 3. Install
  /plugin install cc100x@romiluz13
  ```
- Workflows: Review Arena, Bug Court, Pair Build
- Configuration
- License

#### 10.2 package.json (optional, for npm)
```json
{
  "name": "cc100x",
  "version": "0.1.0",
  "description": "Next-gen orchestration on Agent Teams",
  "author": "Rom Iluz",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/romiluz13/cc100x"
  }
}
```

#### 10.3 GitHub repo creation
```bash
gh repo create romiluz13/cc100x --public --description "Next-gen orchestration built on Claude Code Agent Teams"
git remote add origin https://github.com/romiluz13/cc100x.git
git add -A
git commit -m "Initial CC100x: Agent Teams orchestration"
git push -u origin main
```

---

## Phase 11: Testing & Dogfooding

**Goal**: Test each workflow against a real codebase.

### Test 1: Review Arena
```
# In any project with code to review:
"Review the src/ directory using CC100x Review Arena"
```
Expected: 3 reviewer teammates spawn, review independently, challenge each other, produce unified findings.

### Test 2: Bug Court
```
# In a project with a known bug:
"Debug the authentication failure using CC100x Bug Court"
```
Expected: 3+ investigators spawn with different hypotheses, debate, converge on root cause, fix with TDD.

### Test 3: Pair Build
```
# With a feature to implement:
"Build a user profile page using CC100x Pair Build"
```
Expected: Builder + Live Reviewer spawn, builder gets real-time feedback, hunter joins after, verifier runs tests.

### Success criteria:
- [ ] Agent Teams spawn correctly for each workflow
- [ ] Teammates communicate via messaging
- [ ] Router Contracts are output by all teammates
- [ ] Memory is persisted at workflow end
- [ ] Lead stays in delegate mode (doesn't implement)
- [ ] Conflicts between reviewers are surfaced to user

---

## Build Order Summary

| Phase | What | Depends On |
|-------|------|-----------|
| 0 | Plugin scaffold (manifest, folders, git) | Nothing |
| 1 | cc100x-lead skill (the brain) | Phase 0 |
| 2 | Router Contract skill | Phase 0 |
| 3 | Session Memory skill | Phase 0 |
| 4 | Verification skill | Phase 0 |
| 5 | Review Arena (protocol + 3 reviewer agents) | Phases 1-4 |
| 6 | Bug Court (protocol + investigator agent) | Phases 1-4 |
| 7 | Pair Build (protocol + builder, live-reviewer, hunter, verifier agents) | Phases 1-4 |
| 8 | Planner agent | Phases 1-4 |
| 9 | User CLAUDE.md | Phase 1 |
| 10 | README + GitHub + publish | All above |
| 11 | Testing & dogfooding | All above |

---

## Key Rules

1. **Every agent outputs a Router Contract** (YAML at end of output)
2. **Every agent reads memory first** (`.claude/cc100x/`)
3. **Lead always uses delegate mode** (never implements code)
4. **Teammates own file sets** (no two teammates edit the same file)
5. **READ-ONLY agents** (reviewers, hunter, verifier) include Memory Notes for lead to persist
6. **WRITE agents** (builder, investigator, planner) update memory directly
7. **Agent Teams convention**: use `agents/` folder (not `teammates/`) per Claude Code plugin spec
8. **All skill references**: use `cc100x:` prefix (not `cc10x:`)
