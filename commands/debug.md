---
description: Systematic debugging. Activates DEBUG mode for methodical problem investigation.
argument-hint: <symptom or error>
tier: MEDIUM
tier-rationale: Single debugger agent + systematic-debugging skill; reads files, writes 0–2 fixes.
estimated-tokens: "15k–40k"
risk: Cross-layer bugs (frontend ↔ backend) can pull in a second specialist and push toward HEAVY.
---

# /debug — Systematic Problem Investigation

$ARGUMENTS

---

## Purpose

Systematically investigate issues, errors, or unexpected behavior. Load the `systematic-debugging` skill for deep protocols. Consider delegating to the `debugger` agent.

---

## Behavior

1. **Gather information**
   - Error message
   - Reproduction steps
   - Expected vs actual behavior
   - Recent changes (`git log`, `git diff`)

2. **Form hypotheses**
   - List possible causes
   - Order by likelihood

3. **Investigate systematically**
   - Test each hypothesis
   - Check logs, data flow, state
   - Use elimination method

4. **Fix and prevent**
   - Apply fix
   - Explain root cause
   - Add prevention (test, validation, assertion)

---

## Output Format

```markdown
## 🔍 Debug: [Issue]

### 1. Symptom
[What's happening]

### 2. Information Gathered
- Error: `[error message]`
- File: `[filepath:line]`

### 3. Hypotheses
1. ❓ [Most likely cause]
2. ❓ [Second possibility]

### 4. Investigation
**Testing hypothesis 1:**
[What I checked] → [Result]

### 5. Root Cause
🎯 [Why this happened]

### 6. Fix
```<lang>
// Before
[broken code]

// After
[fixed code]
```

### 7. Prevention
🛡️ [Test / assertion / validation added]
```

---

## Examples

```
/debug login not working
/debug API returns 500 on POST /users
/debug form doesn't submit
/debug async DB query hangs under load
/debug Alembic migration fails on NOT NULL
```

---

## Key Principles

- **Ask before assuming** — get full error context
- **Test hypotheses** — don't guess randomly
- **Explain why** — not just what to fix
- **Prevent recurrence** — add a test or guard
