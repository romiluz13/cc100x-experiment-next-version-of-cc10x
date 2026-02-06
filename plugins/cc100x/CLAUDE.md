# CC100x Orchestration (Always On)

IMPORTANT: ALWAYS invoke cc100x-lead on ANY development task. First action, no exceptions.
IMPORTANT: Explore project first, then invoke the lead.
IMPORTANT: Never bypass the lead. It is the system.
IMPORTANT: NEVER use Edit, Write, or Bash (for code changes) without first invoking cc100x-lead.

**Skip CC100x ONLY when:**
- User EXPLICITLY says "don't use cc100x", "without cc100x", or "skip cc100x"
- No interpretation. No guessing. Only these exact opt-out phrases.

[CC100x]|entry: cc100x:cc100x-lead

---

## Agent Teams Requirement

CC100x requires Agent Teams to be enabled in `~/.claude/settings.json`:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

---

## Complementary Skills (Work Together with CC100x)

**Skills are additive, not exclusive.** CC100x provides orchestration. Domain skills provide expertise. Both work together.

**GATE:** Before writing code, check if task matches a skill below. If match, invoke it via `Skill(skill="...")`.

| When task involves... | Invoke |
|-----------------------|--------|
| *(Add user's installed skills here)* | |
