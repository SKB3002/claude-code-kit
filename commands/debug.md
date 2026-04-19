---
description: Systematic debugging. Activates DEBUG mode for methodical problem investigation.
argument-hint: <symptom or error>
tier: MEDIUM
tier-rationale: Single debugger agent + systematic-debugging skill; reads files, writes 0–2 fixes.
estimated-tokens: "15k–40k"
risk: Cross-layer bugs (frontend ↔ backend) can pull in a second specialist and push toward HEAVY.
---

# /kit:debug — Systematic Problem Investigation

$ARGUMENTS

---

## Flow

**Step 1 — Parse bypass flag.**
If `$ARGUMENTS` begins with `--yes` or `-y` (whitespace-separated), set `bypass = true` and strip the flag from `$ARGUMENTS` before continuing. Otherwise `bypass = false`.

**Step 2 — Load the approval-gate skill.**
Read `skills/approval-gate/SKILL.md` for the gate contract.

**Step 3 — Compute the plan.**
- Planned agent: `kit:debugger`
- Planned skills: `kit:systematic-debugging`, `kit:clean-code`
- Tier: MEDIUM (from this file's frontmatter)

**Step 4 — Render the MEDIUM gate (skip if `bypass`).**

```
⚖️  /kit:debug "<stripped args>"
    → kit:debugger  (+ kit:systematic-debugging, kit:clean-code)
    Tier: MEDIUM · 15k–40k tokens · writes 0–2 files
    Proceed? (y/n/tweak)
```

Parse the user's reply per §3.4 of the approval-gate skill:
- `y`/`yes`/`go` → continue to Step 5
- `n`/`cancel` → append a cancelled-run entry to `.kit/usage.json` and stop. Print: `🚫 Cancelled. No tokens spent beyond this gate.`
- `tweak` → ask the user one clarifying question about scope or symptom, then re-render the gate
- Anything else → re-prompt once with the valid replies listed

**Step 5 — Dispatch.**

```
Agent(
  subagent_type="kit:debugger",
  description="Debug: <short symptom>",
  prompt="<full $ARGUMENTS with context the user supplied>\n\nFollow the systematic-debugging skill. Report root cause, fix, and prevention."
)
```

**Step 6 — Append to the usage log.**
Record to `.kit/usage.json` per §5 of the approval-gate skill. Minimum fields: command, args, tier_declared=MEDIUM, tier_observed (computed from agent response size via `tiers.md` formula), agents=[{name: "kit:debugger", approx_tokens}], skills consumed, files touched, duration, approved=true, chosen_alternative="a".

**Step 7 — Print the inline ledger.**

```
📒  /kit:debug ledger
Ran: kit:debugger
Skills: kit:systematic-debugging, kit:clean-code
Files changed: <N>
Approximate tokens: ~<N>k
Tier declared: MEDIUM (15k–40k) · observed: ~<N>k (<in-tier ✓ | drift ✗>) · duration: <Xm Ys>
Logged to .kit/usage.json (<run-id>)
```

---

## Key principles (inherited by the dispatched agent)

- **Ask before assuming** — get full error context before forming hypotheses
- **Test hypotheses** — don't guess randomly, eliminate systematically
- **Explain why** — root cause, not just the line that changed
- **Prevent recurrence** — add a test, assertion, or guard

The debugger agent's own instructions enforce these. This command file only handles dispatch + accounting.

---

## Examples

```
/kit:debug login not working
/kit:debug API returns 500 on POST /users
/kit:debug form doesn't submit
/kit:debug async DB query hangs under load
/kit:debug -y Alembic migration fails on NOT NULL   (bypass gate)
```
