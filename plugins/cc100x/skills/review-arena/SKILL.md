---
name: review-arena
description: "Multi-perspective adversarial code review using Agent Teams. Spawn 3 specialized reviewers who challenge each other's findings."
---

# Review Arena Protocol

## Overview

Review Arena uses 3 specialized reviewer teammates who first review independently, then challenge each other's findings via peer messaging, producing a unified verdict that's stronger than any single review.

---

## Team Composition

| Teammate | Role | Focus Areas | Mode |
|----------|------|-------------|------|
| **security-reviewer** | Security specialist | Auth, injection, secrets, OWASP top 10, XSS, CSRF | READ-ONLY |
| **performance-reviewer** | Performance specialist | N+1 queries, memory leaks, bundle size, caching, loops | READ-ONLY |
| **quality-reviewer** | Quality specialist | Patterns, naming, complexity, error handling, tests, duplication | READ-ONLY |

All reviewers are READ-ONLY. They cannot edit files. They output Router Contracts with Memory Notes.

---

## Protocol Phases

### Phase 1: Independent Review

Each reviewer works independently on the same target code. No communication during this phase.

**Each reviewer must:**
1. Read memory files (`.claude/cc100x/`)
2. Review target from their specialized perspective
3. Score each finding with confidence (report only >=80)
4. Output their Router Contract

**Lead actions:**
- Assign each reviewer their task from the task list
- Wait for all 3 to complete independently

### Phase 2: Challenge Round

After all 3 reviewers complete, lead shares findings and initiates cross-review:

**Lead sends to each reviewer:**
```
"Here are the findings from the other reviewers. Challenge anything you disagree with
or that affects your perspective:

Security Reviewer found: [summary of security findings]
Performance Reviewer found: [summary of performance findings]
Quality Reviewer found: [summary of quality findings]

Message the other reviewers directly to debate any findings."
```

**Expected interactions:**
- "Security reviewer: Your fix for XSS would introduce an N+1 query. Here's why..."
- "Performance reviewer: The caching you suggested would bypass auth checks. Security concern."
- "Quality reviewer: Both of you missed that this function violates SRP. Here's the refactor..."

**Lead actions:**
- Monitor teammate messages
- Allow 2-3 rounds of debate
- Intervene if debate becomes unproductive

### Phase 3: Consensus

Lead collects final verdicts from all reviewers after challenge round.

**Conflict Resolution Rules:**

| Scenario | Resolution |
|----------|------------|
| All 3 agree | Unified verdict (strongest case) |
| 2/3 agree | Majority rules, document dissent with reasoning |
| Security says CRITICAL, others disagree | **Security wins** (conservative principle) |
| Performance vs Quality conflict | Present both to user with trade-off analysis |
| No consensus | Present all 3 perspectives to user, let them decide |

**Severity Precedence:**
- CRITICAL always overrides MAJOR/MINOR
- If ANY reviewer flags CRITICAL → overall verdict = CHANGES_REQUESTED
- BLOCKING=true if any reviewer's CRITICAL_ISSUES > 0

---

## Output: Unified Router Contract

After consensus, lead produces a merged Router Contract:

```yaml
STATUS: APPROVE | CHANGES_REQUESTED
CONFIDENCE: [average of 3 reviewers, weighted by finding severity]
CRITICAL_ISSUES: [sum across all reviewers, deduplicated]
HIGH_ISSUES: [sum across all reviewers, deduplicated]
BLOCKING: [true if any CRITICAL_ISSUES > 0]
REQUIRES_REMEDIATION: [true if CHANGES_REQUESTED]
REMEDIATION_REASON: "Fix critical issues: [merged list from all reviewers]"
MEMORY_NOTES:
  learnings: [merged from all 3 reviewers]
  patterns: [merged from all 3 reviewers]
  verification: ["Review Arena: {STATUS} with {CRITICAL_ISSUES} critical, {HIGH_ISSUES} high"]
```

---

## Deduplication Rules

When merging findings across reviewers:

1. **Same file:line, same issue** → Keep highest severity, merge fix recommendations
2. **Same file:line, different issues** → Keep both (different perspectives on same code)
3. **Conflicting recommendations** → Document both, flag for user decision
4. **Overlapping findings** → Credit the reviewer who found it first, combine evidence

---

## When to Use Review Arena

| Scenario | Use Full Arena? | Alternative |
|----------|-----------------|------------|
| PR review (significant changes) | Yes | - |
| Post-build review | Yes | - |
| Quick check (< 50 lines) | No | Single quality-reviewer |
| Security-critical code only | No | Single security-reviewer |
| Post-remediation re-review | Abbreviated (quality only) | - |

---

## Task Structure

```
CC100X REVIEW: {target}
├── CC100X security-reviewer: Security review of {target}
├── CC100X performance-reviewer: Performance review of {target}
├── CC100X quality-reviewer: Quality review of {target}
├── CC100X Review Arena: Challenge round (blocked by all 3 reviewers)
└── CC100X Memory Update: Persist review learnings (blocked by challenge round)
```
