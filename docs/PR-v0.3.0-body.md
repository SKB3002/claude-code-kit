# v0.3.0 — approval-first dispatch, live discovery, usage tracking

Minor, non-breaking, additive. All v0.2.1 commands continue to work.

## Summary

- **17 slash commands** (was 11) — 6 new: `/kit:help`, `/kit:budget`, `/kit:ledger`, `/kit:context-budget`, `/kit:hookify`, `/kit:instincts`
- **42 skills** (was 40) — 2 new: `kit:approval-gate`, `kit:instincts`; renamed `kit:brainstorming` → `kit:socratic-gate`
- **Approval-first dispatch** — every command declares a tier (LIGHT/MEDIUM/HEAVY); MEDIUM/HEAVY render an explicit gate with planned agents, skills, token estimate, MoSCoW scope, and ≥2 lighter alternatives
- **Explicit agent dispatch** — 8 commands (`/kit:debug`, `/kit:plan`, `/kit:test`, `/kit:brainstorm`, `/kit:enhance`, `/kit:deploy`, `/kit:create`, `/kit:ui-ux-pro-max`) now open with `Agent(subagent_type="kit:<name>", …)` instead of prose delegation
- **Usage tracking** — every run appends to `.kit/usage.json` (gitignored); `/kit:ledger` aggregates by week/agent/skill/tier/command; optional `/kit:budget` enriches the HEAVY gate
- **`KIT_HOOK_PROFILE` env var** — agent-layer contract (`off`/`minimal`/`standard`/`strict`), not real Claude Code hooks
- **Honesty contract** — every token number prefixed `~`, no dollar conversion, no fabricated quota/reset
- **Fixed** — `marketplace.json` dropped top-level `$schema`/`description` keys that caused strict-validation failures on some Claude Code versions

Design stance: **"Build the right thing, on the user's budget, with the user's consent."** Targets $20-plan users where weekly rate limits matter.

See [CHANGELOG.md](CHANGELOG.md#030--2026-04-19) for the full list.

## Test plan

- [ ] `claude --plugin-dir ./claude-code-kit` loads without errors
- [ ] `/kit:help` renders live catalog — 17 commands, 20 agents, 42 skills
- [ ] `/kit:debug "<something>"` first action is an `Agent(subagent_type="kit:debugger", …)` call, not prose
- [ ] HEAVY gate (e.g. `/kit:create`) renders agents / skills / ~tokens / MoSCoW / ≥2 alternatives; cancelling logs a cancelled entry to `.kit/usage.json`
- [ ] `/kit:create --yes …` bypasses gate and still appends a usage log entry
- [ ] `/kit:budget low` writes `~/.kit/budget.json`; next HEAVY gate shows the budget line
- [ ] `/kit:ledger weekly` / `by-agent` / `by-tier` aggregate correctly from recorded runs
- [ ] `claude plugin validate .` passes (marketplace.json strict-validation fix)
- [ ] `/kit:orchestrate <task>` still requires ≥3 agents and 2-phase execution (regression guard)
