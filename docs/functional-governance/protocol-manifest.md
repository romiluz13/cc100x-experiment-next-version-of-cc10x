# CC100x Protocol Manifest (Functional Canon)

Status: canonical
Last validated: 2026-02-09

## Source of Truth

- `plugins/cc100x/skills`
- `plugins/cc100x/agents`

No other location defines runtime behavior.

## Orchestration Entry

- Single entry orchestrator is `cc100x-lead`.
- Decision routing is strict: ERROR > PLAN > REVIEW > BUILD.
- ERROR always wins on ambiguity (for example, "fix the build" routes to DEBUG).

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:4`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:25`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:34`

## SDLC Workflow Canon

- BUILD protocol: Pair Build + Hunter + full Review Arena + Verifier.
- DEBUG protocol: Bug Court + builder fix + full Review Arena + Verifier.
- REVIEW protocol: Review Arena triad + challenge round.
- PLAN protocol: single planner in plan approval mode.

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:40`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:41`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:42`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:43`

## Team Protocol Skills

- BUILD semantics are defined by `pair-build`.
- DEBUG semantics are defined by `bug-court`.
- REVIEW semantics are defined by `review-arena`.
- Contract/gating semantics are defined by `router-contract`.
- Memory semantics are defined by `session-memory`.

Source: `plugins/cc100x/skills/pair-build/SKILL.md:6`
Source: `plugins/cc100x/skills/bug-court/SKILL.md:6`
Source: `plugins/cc100x/skills/review-arena/SKILL.md:6`
Source: `plugins/cc100x/skills/router-contract/SKILL.md:8`
Source: `plugins/cc100x/skills/session-memory/SKILL.md:137`

## Write Ownership

- Builder owns code writes.
- Planner can write plan/memory artifacts.
- Investigator is READ-ONLY and never edits source.
- Reviewers, hunter, verifier, and live-reviewer are READ-ONLY.

Source: `plugins/cc100x/agents/builder.md:15`
Source: `plugins/cc100x/agents/planner.md:15`
Source: `plugins/cc100x/agents/investigator.md:15`
Source: `plugins/cc100x/agents/security-reviewer.md:15`
Source: `plugins/cc100x/agents/performance-reviewer.md:15`
Source: `plugins/cc100x/agents/quality-reviewer.md:15`
Source: `plugins/cc100x/agents/hunter.md:15`
Source: `plugins/cc100x/agents/verifier.md:15`
Source: `plugins/cc100x/agents/live-reviewer.md:15`

## Task DAG + Remediation Canon

- Workflow execution is task-enforced with DAG dependencies.
- Any blocking contract triggers REM-FIX.
- Remediation always re-enters full review + re-hunt before verifier.
- Memory Update task is mandatory for completion.

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:177`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:688`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:733`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:806`

## Agent Teams Constraints Canon

- Delegate mode is mandatory.
- No nested teams.
- No session resumption of teammates.
- One team per session.
- Lead is fixed for team lifetime.
- Permission inheritance occurs at spawn.
- Broadcast should be used sparingly.
- Team shutdown must end with `TeamDelete()`.

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:775`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:908`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:907`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:914`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:915`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:916`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:917`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:979`
