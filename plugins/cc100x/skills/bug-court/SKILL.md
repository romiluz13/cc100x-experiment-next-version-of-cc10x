---
name: bug-court
description: "Competing hypothesis debugging using Agent Teams. Multiple investigators each champion a hypothesis, then debate to find root cause."
---

# Bug Court Protocol

## Overview

Bug Court replaces single-threaded debugging with **competing hypotheses**. Multiple investigators each champion a different root cause hypothesis, gather evidence, then debate each other. The hypothesis that survives cross-examination wins.

---

## Team Composition

| Teammate | Role | Mode |
|----------|------|------|
| **investigator-1** through **investigator-N** | Hypothesis champion | READ+WRITE (can add debug logging, run tests) |

Number of investigators = number of plausible hypotheses (typically 2-5).

---

## Protocol Phases

### Phase 1: Hypothesis Generation (Lead)

Before spawning investigators, the lead generates hypotheses:

1. **Gather initial evidence:**
   - Error message / stack trace
   - Reproduction steps
   - Recent changes (`git log --oneline -10 -- <affected-files>`)
   - Memory patterns (`patterns.md ## Common Gotchas`)

2. **Generate 2-5 hypotheses:**
   ```
   H1: [Most likely cause based on evidence]
   H2: [Alternative cause]
   H3: [Less likely but possible cause]
   ```

3. **Each hypothesis must be:**
   - **Falsifiable**: Can be proven wrong with a specific test
   - **Specific**: "Race condition in auth middleware" not "timing issue"
   - **Testable**: Clear experiment to confirm or deny

### Phase 2: Investigation (Parallel)

Each investigator works independently to test their hypothesis.

**Each investigator must:**
1. Read memory files
2. Gather evidence FOR their hypothesis
3. Gather evidence AGAINST other hypotheses (if accessible)
4. Write a regression test that demonstrates the bug (RED phase)
5. If root cause confirmed, implement minimal fix (GREEN phase)
6. Output Router Contract

**File ownership during investigation:**
- Each investigator works in isolation (forked context)
- Only ONE investigator's changes get applied (the winner)
- The lead decides which set of changes to keep

### Phase 3: Debate (Peer Messaging)

After all investigators complete:

**Lead shares all evidence and initiates debate:**
```
"All investigators have completed their analysis. Here are the findings:

Investigator 1 ({h1}): [evidence summary]
Investigator 2 ({h2}): [evidence summary]
Investigator 3 ({h3}): [evidence summary]

Challenge each other's findings. Message the other investigators directly.
Try to disprove the other hypotheses using your evidence."
```

**Expected interactions:**
- "Investigator 2: Your theory doesn't explain why the bug only occurs with concurrent requests."
- "Investigator 1: I can disprove your hypothesis. Look at the logs from test X - the timing doesn't match."
- "Investigator 3: Both of you are wrong. The stack trace clearly shows the error originates from..."

**Lead monitors debate and allows 2-3 rounds of messaging.**

### Phase 4: Verdict (Lead)

Lead evaluates:

| Criterion | Weight |
|-----------|--------|
| Evidence strength (reproducible test) | Highest |
| Survived cross-examination | High |
| Explains ALL symptoms | High |
| Simplicity (Occam's razor) | Medium |
| Counter-evidence against alternatives | Medium |

**Decision:**
- **Clear winner**: One hypothesis has strong evidence AND survived debate → Proceed to fix
- **Tie**: Two hypotheses both strong → Present both to user for decision
- **All disproved**: No hypothesis survived → Generate new hypotheses (max 2 rounds total)
- **Investigator already fixed it**: If winning investigator implemented fix with TDD evidence → Skip to review

### Phase 5: Fix Implementation

If winning investigator already has a fix with TDD evidence (RED exit=1, GREEN exit=0):
- Apply their changes
- Proceed to review

If no fix yet:
- Assign builder to implement fix based on winning root cause
- Builder must follow TDD: regression test FIRST, then minimal fix

### Phase 6: Review the Fix

Trigger abbreviated Review Arena (quality-reviewer only, unless fix touches security-sensitive code).

### Phase 7: Persist Memory

Lead persists:
- Root cause → `patterns.md ## Common Gotchas`
- Debug journey → `activeContext.md ## Learnings`
- Fix verification → `progress.md ## Verification`

---

## Task Structure

```
CC100X DEBUG: {error_summary}
├── CC100X investigator-1: Test hypothesis - {h1}
├── CC100X investigator-2: Test hypothesis - {h2}
├── CC100X investigator-3: Test hypothesis - {h3}
├── CC100X Bug Court: Debate round (blocked by all investigators)
├── CC100X Fix: Implement winning fix (blocked by debate)
├── CC100X Review: Review the fix (blocked by fix)
└── CC100X Memory Update: Persist debug learnings (blocked by review)
```

---

## Anti-Hardcode Gate (REQUIRED)

Before implementing any fix, investigators must check:

**Variant dimensions to consider (only if relevant to this bug):**
- Locale/i18n, configuration, roles/permissions, platform/runtime
- Time/timezone, data shape, concurrency, network, caching

**Regression test MUST cover at least one non-default variant.**

---

## Debug Attempt Tracking

Investigators record attempts in standardized format:
```
[DEBUG-N]: {what was tried} → {result}
```

This enables:
- Lead counting attempts for escalation decisions
- Memory preservation of investigation path
- Avoiding repeated failed approaches in future sessions
