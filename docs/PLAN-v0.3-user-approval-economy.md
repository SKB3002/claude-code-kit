# PLAN v0.3.x — User Approval Economy

> Companion plan to [PLAN-v0.3-agents-discoverability.md](PLAN-v0.3-agents-discoverability.md). This document replaces Phase 1's "Automatic Agent Dispatch" with an **approval-first** model and adds the supporting discoverability and accounting primitives. Both land together in the v0.3.0 release.

Status: DRAFT — awaiting user approval before implementation.
Branch: `feat/v0.3-agents-discoverability`.
Philosophy: **"Build the right thing, on the user's budget, with the user's consent."**

---

## 1. Why this exists

Most Claude Code Kit users are on the **$20 Claude Pro plan**, not Max or Enterprise. Weekly rate limits hit fast when a user is doing day-job work + side projects on the same account. If a `/kit:*` command silently fans out 3 agents in parallel, the user can lose a productive evening to a 30-second autopilot decision they didn't authorize.

**Design stance (from [agents/product-manager.md](../agents/product-manager.md) + [agents/product-owner.md](../agents/product-owner.md)):**

| Principle (source) | How it applies here |
|---|---|
| "Don't just build it right; build the right thing." (PM) | Before spending tokens, confirm the plan matches the user's intent. |
| MoSCoW scope discipline (PM) | The gate shows MUST vs SHOULD vs COULD for the task so the user can trim. |
| "Align needs with execution, prioritize value." (PO) | Each agent invocation is framed as a value/cost trade-off the user decides. |
| Happy Path + Sad Path (PM) | Estimates are ranges with an explicit "what makes this blow up" line. |
| Intelligent Prioritization (PO) | If budget is tight, the gate proposes a lighter alternative path, not just "yes/no." |

The kit is not just for personal toy projects — it targets professionals who need predictable, defensible tool spend. This plan treats tokens like money, because for the user, they are.

---

## 2. What we can and cannot do (honest inventory)

### We CAN

- **Classify each command into a complexity tier** (LIGHT / MEDIUM / HEAVY) based on how many agents it dispatches, whether they run in parallel, whether it writes files, and skill count.
- **Render a pre-dispatch approval gate** listing: agents, skills, tier, estimated token range, why the tier, and alternatives.
- **Accept a user-declared budget** stored locally (e.g. `.kit/budget.json`, gitignored) — `low`, `medium`, `ok`, or a numeric target.
- **Adapt gate verbosity and default alternatives** based on declared budget (e.g. on `low`, propose the lighter alternative as default).
- **Emit a post-run ledger** summarising which agents/skills actually ran, approximate token share per component, and a "was it worth it?" self-reflection.

### We CANNOT (and must not pretend otherwise)

- **Read the weekly quota or reset timestamp from Claude Code.** Not exposed. Any claim of "you have X tokens left" would be fabrication. The gate will never display a number we don't have.
- **Predict exact token cost.** Model behaviour is non-deterministic. We give ranges with a confidence level.
- **Bypass the user's own Claude Code rate limiter.** If Anthropic throttles, we throttle. We can only prevent *our* overspend.

---

## 3. The Approval Gate

### 3.1 When it fires

| Command tier | Behaviour |
|---|---|
| **LIGHT** (`/kit:status`, `/kit:preview`, `/kit:help`, `/kit:test` without target) | No gate. Runs directly. These are cheap, read-only, and cancelling them wastes more tokens than running them. |
| **MEDIUM** (`/kit:debug`, `/kit:test <target>`, `/kit:brainstorm`, `/kit:plan`) | One-line preview + quick confirmation ("Proceed? y/n/tweak"). |
| **HEAVY** (`/kit:create`, `/kit:enhance`, `/kit:deploy`, `/kit:orchestrate`, `/kit:ui-ux-pro-max`) | Full gate with agents list, skills list, estimated tier, alternatives, MoSCoW breakdown. Requires explicit approval text ("go" / "tweak" / "cancel"). |

Tier is a property of the command, not the input — predictable UX.

### 3.2 Gate output shape (HEAVY example)

```
⚖️  Kit dispatch preview — /kit:create

Task: "todo app with auth and deploy to vercel"

Planned agents (in order):
  kit:product-owner        — scope + AC
  kit:project-planner      — task breakdown + file plan
  kit:frontend-specialist  — UI scaffold
  kit:backend-specialist   — API + db
  kit:devops-engineer      — deploy config

Planned skills: kit:app-builder, kit:socratic-gate, kit:plan-writing,
                kit:nextjs-react-expert, kit:fastapi-expert, kit:database-design

Tier: HEAVY  (~80k–200k tokens, ~6–12 minutes wall-clock)
Why:  5 agents, scaffold + deploy scope, writes 15–40 files.
Risk: may balloon if auth provider isn't specified (asks back-and-forth).

MoSCoW for this task:
  MUST    — scaffold, auth, db schema, one vercel deploy
  SHOULD  — tests, CI
  COULD   — admin panel, email notifications
  WON'T   — analytics, i18n (scope creep guardrail)

Alternatives:
  (a) Proceed as-is (HEAVY).
  (b) MUST-only: skip SHOULD/COULD → ~40–90k tokens (~MEDIUM).
  (c) Plan-only: run kit:project-planner + kit:product-owner, no code
      → ~15–30k tokens (~LIGHT). Review plan, then opt into (a) or (b).

Your budget: medium (from .kit/budget.json)
Recommended: (c) — matches your declared budget and gives you another
gate before the expensive step.

Reply:  go     — proceed with (a)
        b      — proceed with (b)
        c      — proceed with (c)  [recommended]
        tweak  — edit scope / agent list first
        cancel
```

### 3.3 Gate output shape (MEDIUM example)

```
⚖️  /kit:debug "login fails after session refresh"
    → kit:debugger (+ kit:systematic-debugging skill)
    Tier: MEDIUM · ~15–40k tokens · writes 0 files
    Proceed? (y/n/tweak)
```

### 3.4 Approval-bypass flag

Power users who know what they want can pass `--yes` (or `-y`) as the first argument to any `/kit:*` command to skip the gate. The post-run ledger still fires. This is documented explicitly, not hidden.

---

## 4. Cost estimation — the honest version

### 4.1 Tier table (committed in code, not magic)

Each command file declares its tier in frontmatter:

```yaml
---
description: ...
argument-hint: ...
tier: medium   # light | medium | heavy
tier-rationale: >
  Dispatches 1 agent (kit:debugger), 1 skill, reads files,
  writes at most 1-2 files. No parallel fanout.
estimated-tokens: "15k–40k"
---
```

`/kit:help <command>` surfaces these. `/kit:help` overview shows total if the user ran every command once.

### 4.2 Why tier, not a hard number

Token cost depends on: input size, model, tool-call count, file read sizes, whether the agent asks clarifying questions. We cannot predict. We can bracket. The ranges above are **honest ranges**, chosen from running the kit ourselves during the v0.2.1 functional test — we will document the measurement methodology so users can verify and contribute updates.

### 4.3 What the gate displays

- Tier (LIGHT/MEDIUM/HEAVY) — authoritative, from frontmatter.
- Token range — descriptive, from frontmatter.
- Wall-clock estimate — descriptive, from frontmatter.
- "Why this tier" — 1 sentence, from frontmatter.
- "Risk" — 1 sentence, from frontmatter, names the most common way this command spikes past its tier.

Nothing is computed at runtime except the tier lookup. No magic, no hidden models, no false precision.

---

## 5. Budget-aware adaptation

### 5.1 `.kit/budget.json` — fully optional

```json
{
  "level": "low",
  "weekly_reset_day": "monday",
  "notes": "side projects only; day-job on a separate account"
}
```

**Core rule: if the file doesn't exist, nothing happens differently, no prompt, no nag.** The gate still fires based on the command's tier (HEAVY still asks, LIGHT still skips) — the user just doesn't see any budget-derived lines. We never auto-create this file. We never say "hey, want to set a budget?" unsolicited. The file is a pure opt-in that *enriches* the gate when present.

| Level | Gate behaviour | Default alternative |
|---|---|---|
| `low` | Always propose Plan-only / MUST-only alt. Warn on HEAVY. Show budget line in gate. | (c) Plan-only |
| `medium` | Show alternatives, show budget line, default to as-is. | (a) Proceed |
| `ok` | Minimal gate on HEAVY (tier + confirm). Show budget line. | (a) Proceed |
| *(file absent)* | **Gate runs exactly as if budget were irrelevant.** No budget line, no alternative ranking bias, no nudge to set one. | (a) Proceed |

The only place the user learns the feature exists is `/kit:help budget` and a single line in the README. Discoverable, never intrusive.

### 5.2 `/kit:budget` command (new, LIGHT) — opt-in surface

The command exists only for users who actively *want* a budget feature. Nothing in the kit calls it, suggests it, or blocks on it. If the user never types `/kit:budget`, they never know it exists except via README and `/kit:help`.

```
/kit:budget                 — show current level and notes (or "no budget set")
/kit:budget low|medium|ok   — set level (creates .kit/budget.json if absent)
/kit:budget clear           — remove the file (back to "unset" state)
```

This command writes/reads `.kit/budget.json`. Never phones home. Never tries to scrape Claude CLI state. Removed the earlier `remind-me` sub-command — any nudging violates the "don't force setup" rule.

### 5.3 Weekly reset awareness — what we actually do

We **can't** know the user's real reset timestamp, so:

- `.kit/budget.json` optionally stores `weekly_reset_day` (string, e.g. `"monday"`).
- Heavy-command gates show: *"Your declared reset day is Monday — today is Thursday. 3 days to reset."* — pure arithmetic on the user's own input, no fabrication.
- The user edits this file whenever their plan changes. Zero magic.

---

## 6. Post-run ledger + persistent usage tracking

Every `/kit:*` invocation — gated or not — appends one entry to a single project-scoped JSON file. That file is the source of truth for everything the user wants to understand about their kit spend in this repo.

### 6.1 The file: `.kit/usage.json`

- **Location:** project root, auto-created on first run. Not home-scoped — each repo gets its own usage history because "how much did I burn on *this* side project" is what people actually want to know.
- **Gitignored.** Never committed, never leaves the machine, never sent to Anthropic or anyone else.
- **Single JSON, append-only `runs` array.** Aggregates are computed on read, never cached, so we don't fight bugs where the aggregates drift from the raw data.
- **Self-explaining.** The file starts with a `_doc` key describing the schema so a user opening it in any editor understands it without reading our docs.

### 6.2 Schema

```json
{
  "_doc": "Claude Code Kit usage log. Approximate token counts (derived from response sizes, NOT Anthropic billing). Safe to delete at any time. See /kit:ledger.",
  "version": 1,
  "created_at": "2026-04-19T10:12:00Z",
  "runs": [
    {
      "id": "r_2026-04-19_001",
      "started_at": "2026-04-19T14:22:03Z",
      "ended_at": "2026-04-19T14:28:55Z",
      "duration_seconds": 412,
      "command": "/kit:create",
      "args": "todo app with auth",
      "tier_declared": "HEAVY",
      "tier_observed": "HEAVY",
      "approved": true,
      "chosen_alternative": "a",
      "agents": [
        { "name": "kit:project-planner",      "approx_tokens": 18000 },
        { "name": "kit:frontend-specialist",  "approx_tokens": 34000 },
        { "name": "kit:backend-specialist",   "approx_tokens": 26000 },
        { "name": "kit:devops-engineer",      "approx_tokens": 0, "skipped": true }
      ],
      "skills": [
        "kit:app-builder", "kit:socratic-gate", "kit:plan-writing",
        "kit:nextjs-react-expert", "kit:database-design"
      ],
      "files_written": 23,
      "files_changed": 4,
      "approx_total_tokens": 145000,
      "user_verdict": null,
      "notes": null
    }
  ]
}
```

Every entry carries date, command, tier (declared + observed), agents with per-agent approximate tokens, skills, files touched, and an open slot for the user's later verdict. No aggregates in the file itself — they're derived.

### 6.3 How tokens get estimated

Claude Code does not expose real billing numbers to plugins. We approximate using character count of each `Agent(subagent_type=...)` response body divided by 4 (rough chars-per-token), plus a fixed overhead per tool-call round. This is documented in `skills/approval-gate/tiers.md` with the exact formula and a worked example, so users can sanity-check and contribute corrections.

**Honesty displays:**
- Every token number shown to the user is prefixed with `~`
- The schema `_doc` says "approximate"
- `/kit:ledger` header says "approximate; not Anthropic billing"
- `/kit:help usage` explains the formula

We never round in a way that looks authoritative. `~145,000` not `145,217`.

### 6.4 The inline ledger (prints right after a command finishes)

```
📒  /kit:create ledger
Ran: 4 of 5 planned agents (kit:devops-engineer skipped — deploy cancelled)
Skills: kit:app-builder, kit:socratic-gate, kit:plan-writing, kit:nextjs-react-expert, kit:database-design
Files written: 23
Approximate token share:
  kit:project-planner     22%   (~18k)
  kit:frontend-specialist 41%   (~34k)
  kit:backend-specialist  31%   (~26k)
  other (tooling, reads)   6%   (~5k)
Tier declared: HEAVY (80k–200k) · observed: ~145k (in-tier ✓) · duration: 6m 52s
Logged to .kit/usage.json (r_2026-04-19_001)
Worth it? — you now have a scaffolded Next.js + FastAPI + Postgres app with auth.
    Run `/kit:ledger verdict r_2026-04-19_001 useful` later if this was worth the spend.
Next suggested: /kit:preview start (LIGHT)
```

### 6.5 `/kit:ledger` command (new, LIGHT) — all views read `.kit/usage.json`

```
/kit:ledger                              — last 5 runs, most recent first
/kit:ledger weekly                       — current ISO week summary, totals by tier + by command
/kit:ledger week <YYYY-Www>              — specific ISO week (e.g. 2026-W16)
/kit:ledger by-agent                     — ranked agents across all history with total ~tokens + run count
/kit:ledger by-skill                     — ranked skills same way
/kit:ledger by-tier                      — totals for LIGHT / MEDIUM / HEAVY
/kit:ledger command <name>               — last 10 runs of one command + per-run drift from tier
/kit:ledger roi                          — ROI view, see §6.6
/kit:ledger verdict <id> <useful|wasted|partial> [note]
                                         — tag a specific run after the fact
/kit:ledger clear                        — wipe .kit/usage.json (confirmation required)
```

Retention: keep everything indefinitely by default. File is pure text, tiny, and the user can `clear` whenever. No automatic rotation — rotating without asking is exactly the kind of silent behaviour we're trying to avoid.

### 6.6 Weekly view example

```
📅  /kit:ledger weekly — 2026-W16 (Apr 13–19)

Runs: 34 total
  LIGHT   18 (52%)  — ~62k tokens · avg ~3.4k
  MEDIUM  11 (32%)  — ~310k tokens · avg ~28k
  HEAVY    5 (14%)  — ~720k tokens · avg ~144k

Top agents (by ~tokens): frontend-specialist (28%), project-planner (19%), debugger (14%)
Top skills: socratic-gate (used 21x), plan-writing (12x), nextjs-react-expert (9x)

Cancelled at gate: 3 runs (~estimated saved: ~350k tokens)
Tier drift: 2 runs exceeded declared tier (both /kit:enhance → consider raising to HEAVY)

Compared to last week (2026-W15): +12% total spend, +4 HEAVY runs.
```

### 6.7 ROI — honest framing

Real ROI needs a "value" number only the user can provide. We do not fabricate one. `/kit:ledger roi` shows what we actually know:

```
💰  /kit:ledger roi (all time in this project)

Total approximate tokens: ~1.9M across 128 runs since 2026-04-12
Runs you tagged "useful":    94  (73%) — ~1.3M tokens
Runs you tagged "wasted":     8  ( 6%) — ~180k tokens
Runs you tagged "partial":   11  ( 9%) — ~220k tokens
Untagged:                    15  (12%) — ~200k tokens

Most-useful-per-token command:  /kit:debug       (92% useful, avg ~18k)
Least-useful-per-token command: /kit:ui-ux-pro-max (40% useful, avg ~210k)
Most-cancelled-at-gate:         /kit:create      (4 cancels in 9 approvals = caution flag)

Interpretation is yours. We don't assign dollar values or suggest you should
stop using a command. This is raw signal, not a verdict.
```

The "useful / wasted / partial" verdicts come from the user running `/kit:ledger verdict r_2026-04-19_001 useful` — never auto-assigned. Untagged runs stay untagged.

### 6.8 Privacy + deletability

- File lives only in the project dir
- No network calls, ever
- `.gitignore` entry added in Phase 1e so it's never accidentally committed
- `/kit:ledger clear` wipes it with one command; the user can also just `rm .kit/usage.json` — we document this explicitly so the user knows the escape hatch is trivial

---

## 7. How this changes the v0.3 plan

The existing [PLAN-v0.3-agents-discoverability.md](PLAN-v0.3-agents-discoverability.md) needs these edits:

1. **Pillar 2 rename.** "Automatic Agent Dispatch" → "Approval-First Agent Dispatch." Every command that previously described its agents now *proposes* them via the gate defined here.
2. **Phase 1 redo.** The 8-command rewrite now has two deliverables per command: (a) real `Agent(subagent_type=…)` wiring, (b) the gate metadata in frontmatter (`tier`, `tier-rationale`, `estimated-tokens`).
3. **Command count cap.** The plan caps at ≤15 total. This document adds 3 (`/kit:budget`, `/kit:ledger`, the already-planned `/kit:help`). We remain under the cap. Net count target: 14.
4. **Discoverability integration.** `/kit:help` renders the tier + estimated-tokens from frontmatter automatically — it becomes the single place users go to comparison-shop commands by cost.
5. **`/kit:instincts` interaction.** The already-planned instincts primitive should learn the user's typical approval answers ("always pick (b) for /kit:create") and pre-select that next time. MoSCoW trimming becomes a promotable instinct.

---

## 8. File-level change plan

**New files:**

- `commands/budget.md` — `/kit:budget`
- `commands/ledger.md` — `/kit:ledger`
- `skills/approval-gate/SKILL.md` — the rendering + prompting logic referenced by every gated command
- `skills/approval-gate/tiers.md` — documented tier taxonomy + examples + measurement methodology
- `docs/PLAN-v0.3-user-approval-economy.md` — this file

**Modified files:**

- Every command in `commands/` gets `tier`, `tier-rationale`, `estimated-tokens` frontmatter
- 8 commands being rewritten in Phase 1 gain the approval-gate flow via the new skill
- [.gitignore](../.gitignore) — add `.kit/`
- [README.md](../README.md) — add a "Budget-aware by design" section
- [KIT_PROTOCOL.md](../KIT_PROTOCOL.md) — new section on the approval gate (P0 rule: no HEAVY command may dispatch without passing the gate)
- [PLAN-v0.3-agents-discoverability.md](PLAN-v0.3-agents-discoverability.md) — add a cross-reference to this doc at top, mark Phase 1 as approval-first

**No changes to:**
- Any agent file (agents remain unaware of budget concerns — keeps the P1/P2 separation clean)
- Validation scripts
- MCP config
- `plugin.json` except version bump

---

## 9. Implementation phases (inserted between existing plan's Phase 1 and Phase 2)

**Phase 1a — Tier metadata (1 commit)**
Add `tier`/`tier-rationale`/`estimated-tokens` frontmatter to all 11 existing commands. No behaviour change. Lets `/kit:help` render the data even before the gate ships.

**Phase 1b — Approval-gate skill (1 commit)**
Create `skills/approval-gate/` with `SKILL.md` + `tiers.md`. Skill is a template + rules for how to render the gate, parse the user's reply, and hand control back.

**Phase 1c — Wire Agent dispatch + gate into 8 commands (8 commits, one per command, as already planned)**
Each commit delivers: real `Agent(subagent_type=...)` call, gate invocation, tested approval flow.

**Phase 1d — /kit:budget + /kit:ledger (2 commits)**
Ship the two new LIGHT commands. Add `.kit/` to `.gitignore`.

**Phase 1e — Docs pass (1 commit)**
Update README, KIT_PROTOCOL, existing v0.3 plan cross-ref.

Then resume existing plan from Phase 2 (`/kit:help`), Phase 3, etc.

---

## 10. Success criteria

- A fresh user running `/kit:create "todo app"` sees the HEAVY gate, picks alternative (c), reviews the plan, and either proceeds or cancels — zero tokens wasted on a misunderstood scope.
- A power user on `ok` budget passes `--yes` to every command and never sees a gate.
- Every command's `tier` in frontmatter matches the observed tier in ≥80% of runs during self-dogfood testing.
- `/kit:ledger` output on a real project matches the user's mental model of what happened (validated by asking 3 testers: "does this feel accurate?").
- Zero occurrences of fabricated numbers (remaining quota, reset timestamps, hard token counts) anywhere in the kit.

---

## 11. Decisions (confirmed by user)

1. **Budget file location:** `~/.kit/budget.json` (home-level) with optional project-level `.kit/budget.json` override.
2. **Usage-log file location:** `.kit/usage.json` in the **project root**, always — per-repo history is the point. Retention: unlimited until the user runs `/kit:ledger clear`.
3. **`--yes` / `-y` bypass:** works on all tiers including HEAVY. Documented with risk note. Rationale: power-user flag, same convention as `apt -y` / `npm --yes`; kit still logs the run to `.kit/usage.json` so nothing is invisible.
4. **Release packaging:** ship discoverability + approval economy + usage tracking as a single **v0.3.0**.
5. **Approval-gate architecture:** pure skill ([skills/approval-gate/SKILL.md](../skills/approval-gate/SKILL.md)) — no new agent. Keeps the 20-agent roster stable and the gate feels like a property of the command, not a new character in the cast.

---

## 12. Explicitly not included

- Claude-API cost scraping — we don't have access and won't pretend.
- Per-user learning models of likely cost — overkill, adds a dependency on stored history the user didn't ask for.
- Cost dashboards / web UI — rejected in the parent v0.3 plan, still rejected here.
- "Emergency stop" mid-agent — Claude Code already lets the user Ctrl-C; we don't need to rebuild that.
- Billing integration with Anthropic API — out of scope.

---

## 13. Next step

**User action required:** review §3 (gate shape), §5 (budget file), and §11 (open questions). Once approved — with answers or overrides to §11 — I will execute Phases 1a–1e on this branch, then resume the parent v0.3 plan.

Nothing ships until you say "go."
