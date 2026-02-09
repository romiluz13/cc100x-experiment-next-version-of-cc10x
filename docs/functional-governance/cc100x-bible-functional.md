# CC100x Bible (Functional-Derived)

Status: canonical-functional
Audience: maintainers, auditors, improvement workflows
Rule: every normative statement must cite a functional source

## Scope

CC100x runtime truth is defined only by:

- `plugins/cc100x/skills`
- `plugins/cc100x/agents`

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:4`

## 1) Entry and Routing

CC100x has a single orchestration entrypoint (`cc100x-lead`) with deterministic routing.

- ERROR signals route to DEBUG and have highest priority.
- PLAN, REVIEW, and BUILD follow in that order.
- Conflict handling is explicit: ERROR wins.

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:25`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:29`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:30`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:31`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:32`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:34`

## 2) SDLC as One System

The lifecycle is integrated and task-enforced:

- BUILD: builder + live loop, then hunter, full review arena, verifier.
- DEBUG: competing investigators, debate, builder fix, full review arena, verifier.
- REVIEW: triad review + challenge merge.
- PLAN: plan approval mode with lead approval handshake.

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:40`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:41`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:42`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:43`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:530`

## 3) Workflow Protocols

Each workflow has a dedicated functional protocol:

- Pair Build protocol governs BUILD.
- Bug Court protocol governs DEBUG.
- Review Arena protocol governs REVIEW and post-fix/post-build review depth.

Source: `plugins/cc100x/skills/pair-build/SKILL.md:6`
Source: `plugins/cc100x/skills/pair-build/SKILL.md:177`
Source: `plugins/cc100x/skills/pair-build/SKILL.md:190`
Source: `plugins/cc100x/skills/bug-court/SKILL.md:6`
Source: `plugins/cc100x/skills/bug-court/SKILL.md:149`
Source: `plugins/cc100x/skills/bug-court/SKILL.md:159`
Source: `plugins/cc100x/skills/review-arena/SKILL.md:6`
Source: `plugins/cc100x/skills/review-arena/SKILL.md:123`

## 4) Contract and Gating

Every teammate outputs a Router Contract that the lead validates before progressing.

- Contract schema is mandatory and machine-readable YAML.
- Agent-specific STATUS values are constrained.
- BLOCKING/REQUIRES_REMEDIATION drives REM-FIX task creation.
- Circuit breaker applies before repeated remediations.

Source: `plugins/cc100x/skills/router-contract/SKILL.md:20`
Source: `plugins/cc100x/skills/router-contract/SKILL.md:48`
Source: `plugins/cc100x/skills/router-contract/SKILL.md:195`
Source: `plugins/cc100x/skills/router-contract/SKILL.md:207`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:688`

## 5) Memory and Continuity

Memory is a protocol, not optional note-taking:

- Lead always loads memory before routing.
- Memory update is task-enforced at workflow end.
- READ-ONLY agents emit Memory Notes; lead persists workflow-final.
- Stable anchors are required for safe memory edits.

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:47`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:285`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:855`
Source: `plugins/cc100x/skills/session-memory/SKILL.md:146`
Source: `plugins/cc100x/skills/session-memory/SKILL.md:300`

## 6) File Ownership and Agent Roles

Write/read boundaries are strict:

- Builder owns code writes.
- Planner may write planning/memory artifacts.
- Investigator is read-only during hypothesis phase.
- Reviewers, hunter, verifier, and live-reviewer are read-only.

Source: `plugins/cc100x/agents/builder.md:15`
Source: `plugins/cc100x/agents/planner.md:15`
Source: `plugins/cc100x/agents/investigator.md:15`
Source: `plugins/cc100x/agents/security-reviewer.md:15`
Source: `plugins/cc100x/agents/performance-reviewer.md:15`
Source: `plugins/cc100x/agents/quality-reviewer.md:15`
Source: `plugins/cc100x/agents/hunter.md:15`
Source: `plugins/cc100x/agents/verifier.md:15`
Source: `plugins/cc100x/agents/live-reviewer.md:15`
Source: `plugins/cc100x/skills/router-contract/SKILL.md:109`
Source: `plugins/cc100x/skills/session-memory/SKILL.md:141`

## 7) Task DAG and Remediation Loop

The orchestration engine is task-graph first:

- Workflow tasks define dependency DAGs.
- Downstream phases only open when blockers complete.
- Any remediation re-enters full review + re-hunt before verifier.
- Completion requires memory task completion.

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:177`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:187`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:733`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:803`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:806`

## 8) Agent Teams Constraints

Constraints are first-class operational rules:

- Delegate mode required.
- No nested teams.
- No `/resume` teammate restoration.
- One team per session.
- Lead remains fixed for team lifetime.
- Permission mode inheritance happens at spawn.
- Shutdown requires approval flow then `TeamDelete()`.

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:775`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:908`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:907`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:914`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:915`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:916`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:979`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:985`

## 9) Release Gates

A workflow is done only if all mandatory gates pass:

- memory loaded
- tasks checked
- tasks created
- team created
- all tasks completed
- contracts validated
- memory updated
- team shutdown

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:962`
Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:975`

## 10) Documentation Rule

If a statement in any bible/design doc conflicts with functional files, functional files win.

Source: `plugins/cc100x/skills/cc100x-lead/SKILL.md:993`
