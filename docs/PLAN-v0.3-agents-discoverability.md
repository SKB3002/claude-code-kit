# PLAN — v0.3 Agents & Discoverability

| Field | Value |
|---|---|
| Version target | `0.3.0` |
| Branch | `feat/v0.3-agents-discoverability` |
| Base | `main` at `3ce8f80` (v0.2.1) |
| Status | Phase 1 shipped — approval-first dispatch, `/kit:budget`, `/kit:ledger` all on the branch. Phase 2+ pending. |
| Author | Suyash Bhatkar |
| Plan created | 2026-04-18 |

> **See also:** [PLAN-v0.3-user-approval-economy.md](PLAN-v0.3-user-approval-economy.md) — the companion plan that replaces Phase 1's "Automatic Agent Dispatch" with an **approval-first** model and adds `/kit:budget` + `/kit:ledger` + the approval-gate skill. Both ship together in v0.3.0.

---

## 1. Goal & Philosophy

**Pledge: Own what we build.** Nothing in this plan is copied from another kit. Every borrowed idea is reworded in our voice, integrated with our [KIT_PROTOCOL.md](../KIT_PROTOCOL.md) hierarchy, and earns its place on merit — not presence.

**v0.3 delivers three pillars, in priority order:**

1. **Discoverability.** If a user can't find a command, it doesn't exist. Add `/kit:help` as the first-stop reference.
2. **Auto-agent-routing.** Every `/kit:<command>` that benefits from a specialist will dispatch via the `Agent` tool automatically — not ask Claude to "apply the persona". Deterministic handoff, not interpretation.
3. **Own the best external ideas.** Four selective adoptions from our research audit of `affaan-m/everything-claude-code` (instincts, hook profiles, context budget, hookify), reworked into our idiom.

**Guardrails we will not violate:**

- The 11-command surface stays ≤15 after v0.3 (not 79+). If we add 4 commands, we defend every one.
- P0/P1/P2 rule priority stays intact. `KIT_PROTOCOL.md` remains the single source of truth.
- No harness adapters for Cursor/Codex/etc. Claude Code only.
- No dashboards, GUIs, or Rust control planes. Markdown + Python only.
- No auto-enabled hooks. Opt-in stays opt-in.

---

## 2. Research Inputs

A full research audit of [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) (v1.10.0, ~160k stars, ~3 months old) produced a tiered ranking. Summary of what we're picking up and what we're leaving.

### Adopting (Tier S — reworded in our voice)

| ECC idea | Our adaptation | Rationale |
|---|---|---|
| `continuous-learning-v2` skill (project-scoped instincts with confidence scoring) | `/kit:instincts` command + `instincts/` skill + opt-in hook | Genuinely novel architecture we have no equivalent for. Captures tool-call patterns, promotes to reusable skills. |
| `ECC_HOOK_PROFILE` env var runtime tuning | `KIT_HOOK_PROFILE={minimal\|standard\|strict}` | Solves our "copy-paste from `hooks.example.json`" burden. |
| `/context-budget` command | `/kit:context-budget` | Surfaces token overhead at logical breakpoints — no equivalent in our kit. |
| `/hookify` conversational hook generator | `/kit:hookify` | "Describe intent → plugin emits JSON" is a clear UX win over manual editing. |

### Evaluating (Tier A — deferred, not in v0.3)

- Install profiles (`core | developer | security | research | full`) — premature at our size (40 skills). Revisit at 60+.
- `/harness-audit` + `harness-optimizer` agent — novel meta-capability. Deferred to v0.4.
- `/skill-create` from git history — interesting, but requires more design work. v0.4.
- PRP 5-stage planning pipeline — our single `/kit:plan` → `PLAN-{slug}.md` is simpler. Revisit if users ask.

### Rejecting (will NOT adopt)

- 79+ command catalogs — decision fatigue. Our lean surface is a feature.
- One-agent-per-language (go-fixer, rust-fixer, kotlin-fixer…) — our `backend-specialist` + language skills is better factoring.
- Dashboards (Tkinter or otherwise), Rust control planes, custom LLM layers — violate "a plugin is just markdown".
- Cross-harness adapters (`.cursor/`, `.trae/`, `.kiro/`) — scope creep.
- Flat ruleset style (`RULES.md` 1.7 KB) — inferior to our `KIT_PROTOCOL.md` tier structure.
- GAN-style 3-tier agents — unproven.
- Auto-enabled hooks via 47 KB shipped `hooks.json` — unsafe default.
- Legacy shim commands (our `/orchestrate` stays canonical, no "legacy" markers).

---

## 3. The Three Pillars of v0.3

### Pillar A — Discoverability via `/kit:help`
One command that answers "what does this kit actually do?" in <5 seconds.

### Pillar B — Auto-Agent-Routing
Every `/kit:<command>` that has a natural specialist owner opens with an explicit `Agent(subagent_type="kit:<agent>")` dispatch. No prose like "apply the X persona" — real tool calls.

### Pillar C — Four targeted adoptions
`/kit:instincts`, `/kit:context-budget`, `/kit:hookify`, and the `KIT_HOOK_PROFILE` env var.

---

## 4. Detailed Design — `/kit:help`

### Purpose

Answer three questions in one command:
1. What commands does this kit ship?
2. What agents are available?
3. How do I invoke the right thing for my task?

### Contract

```
description: List kit commands, agents, and skills. Use to discover capabilities. Accepts optional subject: "commands", "agents", "skills", or a specific name.
argument-hint: [commands|agents|skills|<name>]
```

### Behavior

| `$ARGUMENTS` | Response |
|---|---|
| *empty* | **Overview**: one-line summary of the kit + top-level counts + the 11 canonical commands + "how to drill deeper" footer |
| `commands` | Full table of all `/kit:<command>` entries with descriptions, pulled live from [commands/](../commands/) |
| `agents` | Full table of 20 agents, their `description:` frontmatter, and which skills they declare |
| `skills` | Full table of 40 skills grouped by cluster (quality, backend, frontend, workflow, security, ops) |
| `<specific name>` | Resolve it: if it's a command, print its full body. If an agent, print frontmatter + mission. If a skill, print `SKILL.md` summary. If ambiguous, ask. |

### Implementation principle

`/kit:help` is **not** a static markdown listing. It instructs Claude to glob our folders and render live content, so it self-updates when we add primitives. We never manually sync a "list of commands" doc — the live filesystem is the source of truth.

### Prefix-visibility rule (MANDATORY for all rendering)

Every primitive `/kit:help` prints — command, agent, or skill — must carry the `kit:` prefix in the rendered output:

- Commands: `/kit:<name>` (never bare `<name>`)
- Agents: `kit:<name>` (since that's the `subagent_type` used to dispatch)
- Skills: `kit:<name>` (matches the Skill-tool catalog handle)

**Why:** users should never have to guess whether a capability came from our kit, a Claude Code built-in, or another plugin. The prefix is free provenance. This rule also applies to README, CHANGELOG, agent `description:` fields when they reference kit primitives, and any future user-facing surface.

Pseudo-flow the command file will encode:

```
1. If $ARGUMENTS empty → Read .claude-plugin/plugin.json for version. Glob commands/*.md, agents/*.md, skills/*/SKILL.md for counts. Render overview.
2. If $ARGUMENTS is a subject → Glob the relevant folder. Read each file's frontmatter. Render a sorted table.
3. If $ARGUMENTS names a specific primitive → Grep commands/, agents/, skills/ for it. Read the matching file and print.
```

### Output format (overview mode)

```markdown
# Claude Code Kit — v0.3.0

20 agents · 40 skills · 12 commands · 16 validation scripts
Plugin handle: `/kit:<command>` · Repo: claude-code-kit

## Essentials
- `/kit:help` — this page
- `/kit:plan <desc>` — write a plan, no code
- `/kit:orchestrate <task>` — coordinate ≥3 agents
- `/kit:create <app>` — scaffold new app
- `/kit:enhance <change>` — iterate on existing app
- `/kit:debug <issue>` — 7-step systematic debugging
- `/kit:test [target]` — stack-aware test runner
- `/kit:deploy <target>` — pre-deploy checklist + deploy
- `/kit:preview [start|stop|check]` — dev server
- `/kit:status` — project snapshot
- `/kit:brainstorm <idea>` — structured exploration
- `/kit:ui-ux-pro-max <desc>` — design workflow
- `/kit:context-budget` — NEW in v0.3
- `/kit:hookify <intent>` — NEW in v0.3
- `/kit:instincts <show|promote|clear>` — NEW in v0.3

## Next steps
- `/kit:help agents` — see all 20 specialists
- `/kit:help skills` — see the 40 skill modules
- `/kit:help <name>` — deep-dive any one thing
```

### File to create

- [commands/help.md](../commands/help.md) (**new**) — ~60 lines

### Acceptance

- `/kit:help` in a fresh session prints the overview in one tool-call round
- `/kit:help agents` lists exactly the 20 agents found in `agents/`
- Adding a new command file makes it appear in the next `/kit:help commands` call without editing `help.md`

---

## 5. Detailed Design — Auto-Agent-Routing

### The current reality

Audit of [commands/](../commands/) as of v0.2.1:

| Command | Current agent dispatch | Gap |
|---|---|---|
| `/kit:orchestrate` | ✅ Explicit `Agent(subagent_type="backend-specialist", ...)` calls | None — this is our model |
| `/kit:plan` | ⚠️ Says "uses project-planner agent" in prose | No real Agent tool invocation |
| `/kit:create` | ⚠️ Description mentions 4 agents; body has generic "coordinate" prose | No real dispatch |
| `/kit:debug` | ⚠️ 7-step methodology, no agent call | Should delegate to `debugger` |
| `/kit:enhance` | ⚠️ Prose-only | Should route by change type |
| `/kit:ui-ux-pro-max` | ⚠️ Prose-only | Should dispatch to `frontend-specialist` |
| `/kit:test` | ⚠️ Stack-branching prose | Should dispatch to `test-engineer` |
| `/kit:deploy` | ⚠️ Stack-branching prose | Should dispatch to `devops-engineer` |
| `/kit:brainstorm` | ⚠️ Prose-only | Could dispatch to `project-planner` for feature-level ideas |
| `/kit:preview`, `/kit:status` | N/A | Simple utility commands, no agent needed |

**Conclusion:** 8 of 11 commands need standardization.

### The canonical pattern (from `/kit:orchestrate`)

```python
Agent(
    subagent_type="kit:backend-specialist",
    description="<short description>",
    prompt=f"""
    [Context the agent needs — full task, constraints, files to read.]

    [Explicit scope — what to do, what NOT to do.]

    [Success criteria — how you'll know when done.]
    """
)
```

The `prompt=` block is load-bearing. Parallel agents can't see each other's context (this is already the [`Context Passing MANDATORY`](../commands/orchestrate.md#L82) rule in `/kit:orchestrate` and it must propagate).

### The new rule for v0.3 commands

Every non-trivial `/kit:<command>.md` must have a **top-of-file section** titled **"Automatic Agent Dispatch"** that specifies:

1. **Which agent is the primary** — single `Agent(subagent_type=...)` call that opens the command
2. **Which agents are optional secondary dispatches** — and under what condition
3. **What context gets passed** — templated in the markdown so Claude fills it deterministically

Commands where no agent is needed (`/kit:preview`, `/kit:status`, `/kit:help`) get a section stating **"Direct — no agent dispatch"** so the convention is explicit, not silent.

### Example rewrite — `/kit:debug`

**Before (v0.2.1):**
```markdown
# /debug — Systematic Problem Investigation
## Purpose
7-step methodology for root-cause analysis.
## Behavior
1. Symptom → 2. Info gather → 3. Hypotheses → ...
```

**After (v0.3.0):**
```markdown
# /kit:debug — Systematic Problem Investigation

## Automatic Agent Dispatch
Your FIRST action is:

Agent(
  subagent_type="kit:debugger",
  description="Systematic investigation of: $ARGUMENTS",
  prompt="""
  Issue reported: $ARGUMENTS

  Load the systematic-debugging skill. Run the full 7-step methodology:
  1. Symptom → 2. Info → 3. Hypotheses → 4. Investigation → 5. Root cause → 6. Fix → 7. Prevention.

  Return: a structured Debug Report. Do NOT apply the fix yet — propose it for user approval.
  """
)

After the debugger returns, ask the user to approve the proposed fix before editing.
```

### Files to modify

- [commands/plan.md](../commands/plan.md)
- [commands/create.md](../commands/create.md)
- [commands/debug.md](../commands/debug.md)
- [commands/enhance.md](../commands/enhance.md)
- [commands/test.md](../commands/test.md)
- [commands/deploy.md](../commands/deploy.md)
- [commands/brainstorm.md](../commands/brainstorm.md)
- [commands/ui-ux-pro-max.md](../commands/ui-ux-pro-max.md)

### Files to preserve

- [commands/orchestrate.md](../commands/orchestrate.md) — already canonical
- [commands/preview.md](../commands/preview.md) — no agent needed
- [commands/status.md](../commands/status.md) — no agent needed

### KIT_PROTOCOL.md update

Add **§ 5.1 — Command → Agent dispatch contract** codifying the rule. This bumps P0 coverage so every future contributor knows the convention without reading 11 command files.

### Acceptance

- In a fresh session, running `/kit:debug ECONNRESET in queue worker` triggers exactly one `Agent(subagent_type="kit:debugger", ...)` call as the first action
- `grep -l "Automatic Agent Dispatch" commands/*.md` returns ≥9 files after v0.3 merge
- Existing behavior of `/kit:orchestrate` unchanged (regression guard)

---

## 6. Detailed Design — Four Targeted Adoptions

### 6.1 `/kit:context-budget`

**Purpose:** Surface current session's token overhead and suggest strategic `/compact` points before hitting the 95% auto-compact wall.

**Why ours is different from ECC's version:**
- Scope-aware: factors in our modular skill loading pattern (some contexts load 1 skill, others load 8)
- Emits an actionable recommendation, not just a report: "Compact now — you've loaded 3 agents and can drop skills X, Y since they're no longer in scope"
- Never auto-compacts — user always approves

**Contract:**
```
description: Report current context usage and recommend strategic compaction points. Read-only.
argument-hint: [verbose]
```

**File to create:** [commands/context-budget.md](../commands/context-budget.md) (~40 lines)

---

### 6.2 `/kit:hookify`

**Purpose:** Describe a hook intent in natural language; get a ready-to-paste `hooks.json` snippet that matches our schema.

**Example usage:**
```
/kit:hookify on every Edit to a .py file, run ruff check --fix
```

**Expected output:** A JSON block the user can paste into [hooks/hooks.json](../hooks/hooks.json), plus a reminder about the `hooks` top-level key and the `${CLAUDE_PLUGIN_ROOT}` path convention (both were v0.2.0/v0.2.1 stumbling blocks).

**Guardrails:**
- Warns on security-sensitive hooks (Bash `PreToolUse` hooks that can mutate commands)
- Cross-references our existing [hooks/hooks.example.json](../hooks/hooks.example.json) so duplicates are called out

**File to create:** [commands/hookify.md](../commands/hookify.md) (~50 lines)

---

### 6.3 `KIT_HOOK_PROFILE` environment variable

**Purpose:** Let users pick a hook intensity profile without editing JSON.

**Profiles:**
| Profile | Enables |
|---|---|
| `off` (default) | Nothing. Same as current opt-in-via-copy-paste |
| `minimal` | `lint_runner.py` on PostToolUse for Edit/Write only |
| `standard` | minimal + `security_scan.py` on Edit/Write + `test_runner.py --summary` on Stop |
| `strict` | standard + `PreToolUse` Bash pre-check, `schema_validator.py` on schema edits, `api_validator.py` on route edits |

**Implementation:** A new `hooks/profiles/` directory with `minimal.json`, `standard.json`, `strict.json`. The main [hooks/hooks.json](../hooks/hooks.json) reads `${KIT_HOOK_PROFILE}` and includes the matching profile's contents at load time.

**Catch:** Claude Code hook configs don't support includes/references — hooks.json is read literally. We'll implement this via a tiny `hooks/apply-profile.py` script that users run once to generate hooks.json from a profile, OR (preferred) we document the env var as a **contract Claude reads** — when `KIT_HOOK_PROFILE=standard`, `KIT_PROTOCOL.md` §6 tells Claude to run the equivalent validation scripts manually after tool calls, implementing the profile at the agent layer rather than the hook loader.

**Design open question:** Do we implement this as real Claude Code hooks (requires apply-profile.py) or as agent-layer protocol rules (no script needed, but requires every session's Claude to re-read KIT_PROTOCOL.md)? **Recommendation: start with agent-layer rules for v0.3; move to real hooks only if the agent-layer approach proves unreliable.**

**Files to create:**
- [KIT_PROTOCOL.md §6](../KIT_PROTOCOL.md) new section documenting the env var contract
- [hooks/profiles/README.md](../hooks/profiles/README.md) explaining the profiles (optional scripts ship later)

---

### 6.4 `/kit:instincts` (project-scoped learning)

**Purpose:** Capture patterns Claude observes during a session and promote them to reusable skills. This is the single most novel idea from ECC — we will adopt it but with a rigor they don't have.

**Scope design:**

| Storage | Path | Behavior |
|---|---|---|
| Session instincts | Transient — held in active conversation | Not persisted; asked for via `/kit:instincts show` |
| Project instincts | `<repo>/.kit/instincts.yaml` | Git-tracked, per-project. Only written with explicit `/kit:instincts promote` |
| Global instincts | `~/.claude/kit/instincts.yaml` | Promoted only when the same instinct appears in ≥2 projects at ≥0.8 confidence |

**Instinct schema (YAML):**
```yaml
- id: prefer-httpx-over-requests
  trigger: "writing Python HTTP client code"
  action: "use httpx, not requests"
  confidence: 0.75
  domain: "python-backend"
  evidence:
    - "user replaced requests with httpx in 3 commits: abc123, def456, ghi789"
    - "project pyproject.toml declares httpx as direct dep"
  scope: project
  project_id: "claude-code-kit"
  created: 2026-04-18
  last_seen: 2026-04-18
```

**Contract:**
```
description: Inspect, promote, or clear learned instincts for this project. Captures patterns across sessions.
argument-hint: [show|promote|clear|status]
```

**Subcommands:**
- `/kit:instincts show` — lists current project's instincts
- `/kit:instincts promote` — interactively reviews session observations and promotes high-confidence ones to `project` scope
- `/kit:instincts clear [id]` — removes one or all
- `/kit:instincts status` — counts: session/project/global + top 5 by confidence

**Opt-in discipline:**
- No automatic capture. Instincts are only written when the user runs `/kit:instincts promote` and approves each one
- Git-tracked in the project's repo, so team decisions are visible in PRs
- `~/.claude/kit/instincts.yaml` path is private — no network sync

**Files to create:**
- [commands/instincts.md](../commands/instincts.md) (~70 lines)
- [skills/instincts/SKILL.md](../skills/instincts/SKILL.md) (~120 lines explaining the mental model)
- [skills/instincts/schema.md](../skills/instincts/schema.md) (YAML schema reference)

**Non-goals:**
- No automated pattern extraction from git log (ECC does this; it's noisy and low-precision). User explicitly invokes promote.
- No background Haiku agent watching tool calls. Opt-in, synchronous.
- No LLM inference cost burned on every tool use. Zero overhead when not invoked.

---

## 7. What We Explicitly Reject from ECC

Documented here so future contributors don't re-propose these:

| Rejected idea | Why |
|---|---|
| Grow past 15 commands | Decision fatigue. Their 79+ catalog needs `COMMANDS-QUICK-REF.md` *because* it's unnavigable. |
| 1-agent-per-language (python-reviewer, rust-reviewer, go-reviewer, …) | Our `backend-specialist` + skill modules is better factoring. Less bloat, more depth. |
| Tkinter desktop dashboard | Wrong medium for a CLI plugin. |
| `ecc2/` Rust control plane | Platform drift. Claude Code's built-in Agent tool already handles routing. |
| Custom LLM layer (`src/llm/`) | A plugin is just markdown. |
| Cross-harness adapters (`.cursor/`, `.kiro/`, `.trae/`) | Scope creep. We are a Claude Code plugin, not a polyglot agent framework. |
| Auto-enabled hooks (47 KB shipped `hooks.json`) | Unsafe default. Violates our consent model. |
| "Legacy shim" command pattern | Our `/orchestrate` stays canonical. No gravestones in `commands/`. |
| PRP 5-stage planning (`/prp-prd` → `/prp-plan` → `/prp-implement` → `/prp-commit` → `/prp-pr`) | Our single `/kit:plan` → `PLAN-{slug}.md` is simpler for equivalent outcomes. |
| Flattened `RULES.md` ruleset | Inferior to our `KIT_PROTOCOL.md` P0/P1/P2 tier structure. |
| GAN-style 3-tier agents (generator/evaluator/planner) | Metaphor doesn't carry. |
| 7-language README translations | We ship one language well. |

---

## 8. File-Level Change Plan

### New files (10)

```
commands/help.md
commands/context-budget.md
commands/hookify.md
commands/instincts.md
skills/instincts/SKILL.md
skills/instincts/schema.md
hooks/profiles/README.md
hooks/profiles/minimal.json            (placeholder; generated, not live)
hooks/profiles/standard.json           (placeholder; generated, not live)
hooks/profiles/strict.json             (placeholder; generated, not live)
docs/PLAN-v0.3-agents-discoverability.md  (this file)
```

### Modified files (13)

```
.claude-plugin/plugin.json             (version 0.2.1 → 0.3.0)
.claude-plugin/marketplace.json        (drop $schema + description top-level keys — strict validator fix)
CHANGELOG.md                           (new 0.3.0 section)
README.md                              (mention /kit:help, 4 new commands, no structural changes)
KIT_PROTOCOL.md                        (add §5.1 Command→Agent Dispatch, §6 KIT_HOOK_PROFILE, §10 /kit:help discoverability)
CLAUDE.md                              (add §Auto-agent-routing convention)
commands/plan.md                       (add Automatic Agent Dispatch section)
commands/create.md                     ("")
commands/debug.md                      ("")
commands/enhance.md                    ("")
commands/test.md                       ("")
commands/deploy.md                     ("")
commands/brainstorm.md                 ("")
commands/ui-ux-pro-max.md              ("")
```

### Unchanged (deliberate)

```
agents/**                              all 20 agent files — no rework needed
skills/**                              all 40 skills, except the new instincts skill
commands/orchestrate.md                already canonical
commands/preview.md                    utility, no agent
commands/status.md                     utility, no agent
hooks/hooks.json                       stays empty (opt-in)
hooks/hooks.example.json               stays as reference
.mcp.json / .mcp.example.json          no change
LICENSE, CONTRIBUTING.md               no change
All 16 validation scripts              no change
```

---

## 9. Implementation Phases

Implemented on this branch (`feat/v0.3-agents-discoverability`), one commit per phase for reviewability.

### Phase 0 — Prep (this plan)
- ✅ Create branch `feat/v0.3-agents-discoverability`
- ✅ Commit this plan as `docs/PLAN-v0.3-agents-discoverability.md`
- ✅ Push branch

### Phase 1 — Auto-agent-routing (highest leverage, breaking behavior, no API break)
- Rewrite 8 commands to the `Automatic Agent Dispatch` pattern
- Update `KIT_PROTOCOL.md` §5.1
- Manual test: run each command in a scratch repo via `claude -p`, confirm `Agent(...)` dispatch fires as first action
- Commit: `feat: auto-dispatch agents from /kit commands`

### Phase 2 — `/kit:help`
- Create `commands/help.md`
- Verify live rendering against current filesystem (run `claude -p "/kit:help agents"` and count 20)
- Commit: `feat: /kit:help for live discoverability`

### Phase 3 — Four targeted adoptions
Split into 3 commits:
- 3a: `/kit:context-budget` + skill reference in `llm-observability`
- 3b: `/kit:hookify` + guardrails
- 3c: `/kit:instincts` + new `instincts` skill + `.kit/instincts.yaml` schema

### Phase 4 — Hook profiles (agent-layer implementation)
- New `KIT_PROTOCOL.md §6` documenting `KIT_HOOK_PROFILE` contract
- `hooks/profiles/README.md` explaining profiles
- No script yet — agent-layer interpretation only
- Commit: `feat: KIT_HOOK_PROFILE runtime profiles (agent-layer)`

### Phase 5 — Docs + strict-validation fix
- Drop `$schema` + `description` from `marketplace.json` top level
- Update `CHANGELOG.md` to 0.3.0
- Update `README.md` with new command list + help pointer
- Update `CLAUDE.md` with new auto-routing convention
- Bump `plugin.json` to 0.3.0
- Commit: `chore: docs + strict-validation fix for v0.3.0`

### Phase 6 — Release
- Tag `v0.3.0`
- `claude plugin marketplace update claude-code-kit`
- `claude plugin install kit@claude-code-kit --scope user` (clean re-install test)
- Run functional test — same format as v0.2.1 smoke test: exercise `/kit:help`, `/kit:debug`, `/kit:orchestrate` in `/tmp/kit-smoke-test-v03/` with `claude -p`
- Write GitHub release notes from the CHANGELOG 0.3.0 section
- Merge branch to `main` via PR (not force-push)

---

## 10. Testing Strategy

### Pre-merge validation

| Check | How |
|---|---|
| Manifest valid | `claude plugin validate .` must pass (this also motivates the `$schema`/`description` cleanup in Phase 5) |
| Hook schema valid | `python -m json.tool < hooks/hooks.json` passes + manual install loads cleanly |
| `/kit:help` renders | `claude -p --model haiku "/kit:help"` returns ≤400 words with the 15-command list |
| Agent dispatch fires | `claude -p --model haiku "/kit:debug test error"` log shows 1+ `Agent(subagent_type="kit:debugger", ...)` entry in the first iteration |
| Regression: `/kit:orchestrate` still invokes ≥3 agents | `claude -p "/kit:orchestrate build a todo api"` log count ≥3 `Agent()` calls |
| No broken cross-references | `grep -rn "claude-code-kit:" commands/ agents/ skills/` returns zero (we did this already in v0.2.0) |
| Every new command has frontmatter | `python scripts/validate_frontmatter.py commands/` (we'll write a 30-line script) |

### Post-release monitoring

- Ask two external users (friend + 1 more) to run `claude plugin update kit`, then `/kit:help`, then `/kit:context-budget`. Collect friction.
- Watch GitHub Issues for 7 days before planning v0.3.1.

---

## 11. Release Strategy

### Semver decision

This is **not** a breaking API change from the user's perspective:
- All v0.2.1 commands continue to work
- `/kit:` namespace unchanged
- Agents, skills, scripts all intact

But it **is** a significant feature release:
- +4 new commands (`help`, `context-budget`, `hookify`, `instincts`)
- +1 new skill (`instincts`)
- Agent dispatch becomes deterministic (behavior change, not API change)
- New env var contract (`KIT_HOOK_PROFILE`)

**Semver: `0.3.0`** — minor bump, non-breaking, additive.

### Branch → main flow

1. All work lands on `feat/v0.3-agents-discoverability`
2. Each phase is a separate commit for clean `git log`
3. Open PR to `main` once Phase 6 functional test passes
4. Self-review the PR before merging (no force-push; keep commit history)
5. Merge via squash or rebase-merge — preserve the phase commits if preferred for future reference
6. Tag `v0.3.0` on `main` after merge; push tag
7. Update marketplace cache automatically picks up new version

### Friend upgrade path

```
/plugin marketplace update claude-code-kit
/plugin update kit
# restart Claude Code
/kit:help                # confirm new command list shows
```

---

## 12. Risks & Open Questions

| Risk | Mitigation |
|---|---|
| `Agent()` tool syntax in command `.md` files is Claude Code version-specific. If v2.2 changes it, our commands break. | Commit tests that fire each command via `claude -p` in Phase 6. On any CLI-side breaking change, we update the pattern in `KIT_PROTOCOL.md §5.1` once, and commands inherit it. |
| `KIT_HOOK_PROFILE` agent-layer approach depends on Claude reading `KIT_PROTOCOL.md §6` every session. If context is compacted aggressively, the contract is lost mid-session. | Start with agent-layer; if we see drift, fall back to script-based hooks generation. |
| `/kit:instincts promote` writes to `.kit/instincts.yaml` at repo root, potentially polluting repos. | Document clearly: add `.kit/` to `.gitignore` in CLAUDE.md if privacy is desired. Default is git-tracked on purpose — team visibility is the feature. |
| `/kit:help` dynamic listing fails if folder structure changes | Acceptance tests: Phase 2 verifies counts against `agents/*.md` glob. Any `agents/` reorg triggers a `/kit:help` re-test. |
| The four new commands push us from 11 → 15. Slippery slope toward ECC-style bloat. | Hard cap documented in this plan: **≤15 commands**. Any v0.4+ additions require removing or merging an existing one. |

### Open questions (to decide before Phase 3)

1. **Instinct promotion UX** — synchronous review in terminal (one-at-a-time), or markdown table for batch approval? Lean: batch markdown.
2. **`/kit:context-budget`** — does it call the Anthropic API's token-count endpoint, or estimate locally? Local estimation is free and good enough. Confirm.
3. **Should `/kit:help <name>` for a skill render the full `SKILL.md` or just the frontmatter+first section?** Lean: frontmatter + first H2 section, with a pointer "Read the full skill: `skills/<name>/SKILL.md`".
4. **Do we rename any current command for clarity?** e.g., `/kit:ui-ux-pro-max` is a mouthful — consider `/kit:design`. But any rename is a breaking change from v0.2.1. Lean: keep names, revisit at v1.0.

---

## 13. What v0.3.0 does NOT do

Clarifying non-goals, so scope creep is obvious when it happens:

- Does not add install profiles (Tier A deferred)
- Does not add `/kit:harness-audit` or a `harness-optimizer` agent (Tier A deferred)
- Does not add PRP-style multi-stage planning (rejected)
- Does not change any agent's or skill's content substantively
- Does not add Node.js test harness for scripts
- Does not add multi-language README translations
- Does not touch [.mcp.json](../.mcp.json) / MCP servers story
- Does not add a GitHub Actions CI pipeline (deferred to v0.4)
- Does not attempt VS Code extension compatibility (separate investigation)

---

## 14. Success Definition

v0.3.0 is successful if, three weeks after release:

1. An external user can install, run `/kit:help`, and choose the right command for their task within 30 seconds
2. `/kit:debug <issue>` in a fresh repo reliably dispatches the `debugger` agent via tool call (not prose interpretation) in 10/10 trials
3. A new contributor can add a 16th command by reading `KIT_PROTOCOL.md §5.1` and copying one existing command's structure, without asking questions
4. Zero reports of "I didn't know that existed" for any shipped primitive
5. The plugin loads cleanly on install (`Status: ✔ enabled`) on a clean Claude Code ≥v2.1.113

If any of the above fails, plan a v0.3.1 focused on the specific gap.

---

_End of plan. Review before committing to implementation._
