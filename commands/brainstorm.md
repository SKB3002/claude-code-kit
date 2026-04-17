---
description: Structured brainstorming for projects and features. Explores multiple options before implementation.
argument-hint: <topic to brainstorm>
---

# /brainstorm — Structured Idea Exploration

$ARGUMENTS

---

## Purpose

Activate brainstorm mode to explore options **before** committing to an implementation. Load the `brainstorming` skill for detailed protocols.

---

## Behavior

1. **Understand the goal**
   - What problem are we solving?
   - Who is the user?
   - What constraints exist?

2. **Generate options**
   - At least 3 different approaches
   - Each with pros and cons
   - Consider unconventional solutions

3. **Compare and recommend**
   - Summarize tradeoffs
   - Give a recommendation with reasoning

---

## Output Format

```markdown
## 🧠 Brainstorm: [Topic]

### Context
[Brief problem statement]

---

### Option A: [Name]
[Description]

✅ **Pros:**
- [benefit]

❌ **Cons:**
- [drawback]

📊 **Effort:** Low | Medium | High

---

### Option B: [Name]
…

---

### Option C: [Name]
…

---

## 💡 Recommendation

**Option [X]** because [reasoning].

What direction would you like to explore?
```

---

## Examples

```
/brainstorm authentication system
/brainstorm state management for complex form
/brainstorm database schema for social app
/brainstorm caching strategy for an LLM gateway
```

---

## Key Principles

- **No code** — this is about ideas, not implementation
- **Visual when helpful** — use diagrams for architecture
- **Honest tradeoffs** — don't hide complexity
- **Defer to user** — present options, let them decide
