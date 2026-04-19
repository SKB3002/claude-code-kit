---
description: Create a project plan using the project-planner agent. No code — only plan file generation.
argument-hint: <what to plan>
tier: MEDIUM
tier-rationale: One agent (project-planner), writes a single docs/PLAN-*.md, no code changes.
estimated-tokens: "20k–50k"
risk: Plans for large multi-phase initiatives can approach HEAVY if the planner pulls in many file reads.
---

# /kit:plan — Project Planning Mode

$ARGUMENTS

---

## 🔴 Absolute rules

1. **No code writing.** This command produces a plan file only.
2. **Dynamic naming.** `docs/PLAN-{slug}.md` — slug derived from the request, lowercase, hyphen-separated, ≤ 30 chars.

---

## Flow

**Step 1 — Parse bypass flag.**
If `$ARGUMENTS` starts with `--yes` or `-y`, set `bypass = true` and strip the flag.

**Step 2 — Load the approval-gate skill.**
Read `skills/approval-gate/SKILL.md`.

**Step 3 — Compute the plan.**
- Planned agent: `kit:project-planner`
- Planned skills: `kit:plan-writing`, `kit:socratic-gate`, `kit:clean-code`
- Tier: MEDIUM

**Step 4 — Render the MEDIUM gate (skip if `bypass`).**

```
⚖️  /kit:plan "<stripped args>"
    → kit:project-planner  (+ kit:plan-writing, kit:socratic-gate, kit:clean-code)
    Tier: MEDIUM · 20k–50k tokens · writes 1 file (docs/PLAN-<slug>.md)
    Proceed? (y/n/tweak)
```

Replies per §3.4 of the approval-gate skill. On cancel: append cancelled-run entry to `.kit/usage.json`, print `🚫 Cancelled. No plan written.` and stop.

**Step 5 — Dispatch.**
Derive the slug: extract 2–3 key words → lowercase → hyphen-joined → trim to 30 chars. Then:

```
Agent(
  subagent_type="kit:project-planner",
  description="Plan: <short subject>",
  prompt=<<
    CONTEXT:
    - User request: $ARGUMENTS
    - Mode: PLANNING ONLY — no code.
    - Output: docs/PLAN-<slug>.md

    TASK:
    1. Follow the project-planner's Socratic Gate before writing.
    2. Write docs/PLAN-<slug>.md with: goals, task breakdown, dependencies,
       agent assignments, and a verification checklist.
    3. Do NOT write code files.
    4. Return the exact filename you created.
  >>
)
```

**Step 6 — Write usage log (MANDATORY — use the Write tool, do not skip).**

Path: `.kit/usage.json` in the **user's current working directory** (their project), NOT inside `${CLAUDE_PLUGIN_ROOT}`.

1. Read `.kit/usage.json` if it exists → parse the JSON. If absent → start with `{"runs": []}`.
2. Append one entry to `runs`:
   - `id`: `"r_<YYYY-MM-DD>_<NNN>"` (today + zero-padded seq = existing length + 1)
   - `started_at` / `ended_at`: ISO-8601 UTC, `command`: `"/kit:plan"`, `args`: stripped args
   - `tier_declared`: `"MEDIUM"`, `tier_observed`: recomputed from output size
   - `approved`: `true` (or `false` if cancelled), `chosen_alternative`: `null`
   - `agents`: `[{"name": "kit:project-planner", "approx_tokens": <N>}]`
   - `skills`: `["kit:plan-writing", "kit:socratic-gate", "kit:clean-code"]`
   - `files_written`: 1 (the PLAN-*.md only; reads don't count), `approx_total_tokens`: integer
   - `user_verdict`: `null`, `notes`: null
3. Write the updated JSON back using the **Write tool**.

**Step 7 — Print the inline ledger.**

```
📒  /kit:plan ledger
Ran: kit:project-planner
Skills: kit:plan-writing, kit:socratic-gate, kit:clean-code
Plan file: docs/PLAN-<slug>.md
Approximate tokens: ~<N>k
Tier declared: MEDIUM (20k–50k) · observed: ~<N>k (<in-tier ✓ | drift ✗>) · duration: <Xm Ys>
Logged to .kit/usage.json (<run-id>)
Next suggested: /kit:create  (if greenfield) or /kit:enhance  (if adding to existing app)
```

---

## Naming examples

| Request | Plan file |
|---|---|
| `/kit:plan e-commerce site with cart` | `docs/PLAN-ecommerce-cart.md` |
| `/kit:plan mobile app for fitness`    | `docs/PLAN-fitness-app.md` |
| `/kit:plan add dark mode feature`     | `docs/PLAN-dark-mode.md` |
| `/kit:plan FastAPI rate limiting`     | `docs/PLAN-rate-limit.md` |

---

## Usage

```
/kit:plan e-commerce site with cart
/kit:plan SaaS dashboard with analytics
/kit:plan -y LLM observability integration with Langfuse   (bypass gate)
```
