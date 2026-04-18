---
description: Structured brainstorming for projects and features. Explores multiple options before implementation.
argument-hint: <topic to brainstorm>
tier: MEDIUM
tier-rationale: Socratic questioning with the user — usually 1–2 agent rounds, no file writes.
estimated-tokens: "10k–30k"
risk: Scope creep if the user keeps adding requirements mid-brainstorm; can drift toward HEAVY.
---

# /kit:brainstorm — Structured Idea Exploration

$ARGUMENTS

---

## Flow

**Step 1 — Parse bypass flag.**
If `$ARGUMENTS` starts with `--yes` or `-y`, set `bypass = true` and strip the flag.

**Step 2 — Load the approval-gate skill.**
Read `skills/approval-gate/SKILL.md`.

**Step 3 — Compute the plan.**
- Planned agent: `kit:product-manager` (owns MoSCoW + "build the right thing" framing)
- Planned skills: `kit:socratic-gate`, `kit:brainstorm`/`kit:plan-writing` loaded as reference
- Tier: MEDIUM

**Step 4 — Render the MEDIUM gate (skip if `bypass`).**

```
⚖️  /kit:brainstorm "<stripped args>"
    → kit:product-manager  (+ kit:socratic-gate)
    Tier: MEDIUM · 10k–30k tokens · writes 0 files
    Proceed? (y/n/tweak)
```

Replies per §3.4 of the approval-gate skill. On cancel: append cancelled-run entry to `.kit/usage.json`, print `🚫 Cancelled. No brainstorm run.` and stop.

**Step 5 — Dispatch.**

```
Agent(
  subagent_type="kit:product-manager",
  description="Brainstorm: <short topic>",
  prompt=<<
    TOPIC: <stripped args>

    TASK:
    1. Apply the Socratic Gate: ask 3 clarifying questions before producing options.
       Wait for the user's reply.
    2. Once the user answers, produce at least 3 distinct approaches.
       For each: name, short description, ✅ pros, ❌ cons, effort (Low/Medium/High).
    3. Summarise trade-offs and recommend one — with reasoning.
    4. End with: "What direction would you like to explore?"

    RULES:
    - No code. Ideas only.
    - Use diagrams or tables where they clarify architecture.
    - Be honest about complexity and cost; never hide tradeoffs.
    - Defer to the user on the final direction.
  >>
)
```

**Step 6 — Append to the usage log** per §5 of the approval-gate skill. `files_written: 0`.

**Step 7 — Print the inline ledger.**

```
📒  /kit:brainstorm ledger
Ran: kit:product-manager
Skills: kit:socratic-gate
Files written: 0
Approximate tokens: ~<N>k
Tier declared: MEDIUM (10k–30k) · observed: ~<N>k (<in-tier ✓ | drift ✗>) · duration: <Xm Ys>
Logged to .kit/usage.json (<run-id>)
Next suggested: /kit:plan <slug>   (turn the chosen direction into a concrete plan, MEDIUM)
```

---

## Key principles (inherited by the dispatched agent)

- **No code** — ideas only
- **Honest tradeoffs** — don't hide complexity
- **Defer to user** — present options, let them decide
- **Visual when it helps** — diagrams or tables for architecture trade-offs

---

## Examples

```
/kit:brainstorm authentication system
/kit:brainstorm state management for complex form
/kit:brainstorm database schema for social app
/kit:brainstorm -y caching strategy for an LLM gateway   (bypass gate)
```
