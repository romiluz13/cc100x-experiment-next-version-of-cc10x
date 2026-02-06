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
| **Teammates** | `teammates/*.md` - Spawn prompt templates for each role |
| **Protocols** | `protocols/*.md` - Workflow definitions (how teammates interact) |
| **Memory** | `.claude/cc100x/` - Same 3-file system as CC10x |
| **Router Contract** | Same YAML format, output by all teammates |

---

## Reference Materials

All reference files are in `reference/`:
- `agent-teams-complete-docs.md` - Full Agent Teams documentation
- `cc10x-orchestration-bible.md` - CC10x architecture (what we evolved from)
- `cc10x-router-SKILL.md` - CC10x router logic
- `*.md` (agent files) - CC10x agent definitions
- `anthropic-2026-features-research.md` - All Anthropic 2026 features
