---
description: List kit commands, agents, and skills. Use to discover capabilities. Accepts optional subject: "commands", "agents", "skills", or a specific name.
argument-hint: [commands|agents|skills|<name>]
tier: LIGHT
tier-rationale: Read-only glob + frontmatter extraction from ${CLAUDE_PLUGIN_ROOT}. No agents, no writes, no network.
estimated-tokens: "<5k"
risk: None — pure directory scan.
---

# /kit:help — Live capability index

$ARGUMENTS

> All output below is rendered from the **live filesystem** at `${CLAUDE_PLUGIN_ROOT}`. Adding a command/agent/skill file makes it appear here automatically. Do not hardcode lists — always glob.

---

## Prefix-visibility rule (MANDATORY)

Every primitive rendered by `/kit:help` carries the `kit:` prefix:
- Commands: `/kit:<name>` (never bare `<name>`)
- Agents: `kit:<name>` (the `subagent_type` used to dispatch)
- Skills: `kit:<name>` (the Skill-tool catalog handle)

This matches the README-wide convention. Never strip the prefix, even when it looks redundant.

---

## Flow

**Step 1 — Classify the argument.**

| `$ARGUMENTS` (trimmed, lowercased) | Mode |
|---|---|
| *empty*              | OVERVIEW |
| `commands` / `cmd`   | LIST-COMMANDS |
| `agents` / `agent`   | LIST-AGENTS |
| `skills` / `skill`   | LIST-SKILLS |
| anything else        | LOOKUP — resolve a specific primitive name |

**Step 2 — Render per mode.**

### OVERVIEW

1. Read `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json` → extract `version`.
2. Glob: `commands/*.md`, `agents/*.md`, `skills/*/SKILL.md`. Count each.
3. Glob `skills/*/scripts/*.py` for validation-script count.
4. Render:

```
# Claude Code Kit — v<version>

<N agents> · <N skills> · <N commands> · <N validation scripts>
Plugin handle: `/kit:<command>` · Repo: claude-code-kit

## Commands (by tier)

LIGHT   <list of /kit:<name> commands with tier: LIGHT, comma-separated>
MEDIUM  <list of /kit:<name> commands with tier: MEDIUM, comma-separated>
HEAVY   <list of /kit:<name> commands with tier: HEAVY, comma-separated>

MEDIUM and HEAVY commands render an approval gate before dispatching agents.
LIGHT commands run directly. Pass `--yes` or `-y` to bypass any gate.

## Drill deeper

- /kit:help commands         — full table with descriptions + estimated-tokens
- /kit:help agents           — all 20 specialists
- /kit:help skills           — all 41 skill modules grouped by cluster
- /kit:help <name>           — deep-dive any one thing
- /kit:ledger weekly         — see what you've actually spent
- /kit:budget [low|medium|ok] — set optional budget (home-level)
```

Tier buckets come from each command's frontmatter. Never hardcode the tier of a command — read it.

### LIST-COMMANDS

For each file in `commands/*.md`:
1. Read frontmatter. Extract: `description`, `argument-hint`, `tier`, `estimated-tokens`.
2. Sort alphabetically.
3. Render as a markdown table:

```
| Command | Tier | ~tokens | Purpose |
|---|---|---|---|
| /kit:brainstorm <idea> | MEDIUM | 10k–30k | Structured exploration, 3 options with trade-offs |
| /kit:budget [low\|medium\|ok\|clear] | LIGHT | <2k | Opt-in budget declaration |
| ...
```

Use each command's `argument-hint` to assemble the left column (e.g. `/kit:brainstorm <idea>`). Never fabricate an argument hint.

Footer: `Run /kit:help <name> for the full body of any command.`

### LIST-AGENTS

For each file in `agents/*.md`:
1. Read frontmatter. Extract: `name`, `description`, `skills` (list).
2. Sort by the natural grouping from KIT_PROTOCOL.md §7 (Architects / Backend-data-infra / Frontend-UX / Quality-ops).
3. Render four tables, one per group:

```
### <Group name>

| Agent | Purpose | Skills loaded |
|---|---|---|
| kit:<name> | <description, first sentence> | kit:<skill>, kit:<skill>, … |
```

If you can't determine the group from frontmatter, fall back to the roster in [KIT_PROTOCOL.md §7](../KIT_PROTOCOL.md) — read that file and use its groupings.

### LIST-SKILLS

For each `skills/*/SKILL.md`:
1. Read frontmatter. Extract: `name`, `description`.
2. Group by the clusters listed in README.md (Cross-stack quality, Backend/Python, LLM/AI, Frontend, Workflow, Security, Ops). Group-membership heuristic:
   - Read README.md once; find which cluster the skill name appears in
   - Skills not in any cluster → bucket them under "Other"
3. Render one table per cluster (only clusters with ≥1 hit).

Footer: `Run /kit:help <skill-name> to see that skill's full SKILL.md.`

### LOOKUP `<name>`

Resolution order:
1. Strip optional `kit:` or `/kit:` prefix from the argument.
2. Search `commands/<name>.md`. If found → print the full body with a 2-line header (`## /kit:<name>` + tier/tokens). Return.
3. Search `agents/<name>.md`. If found → print frontmatter + the first section of the body (until the next `##`). Return.
4. Search `skills/<name>/SKILL.md`. If found → print frontmatter + first H2 section. Footer: `Read the full skill: skills/<name>/SKILL.md`. Return.
5. If nothing matches, run a case-insensitive grep against `commands/`, `agents/`, `skills/` for the argument. If there's a single fuzzy match, ask: `Did you mean /kit:<name>? (y/n)`. If multiple, list top 5.
6. If still nothing, print: `No kit primitive found matching "<arg>". Run /kit:help to see what's available.`

**Step 3 — Done.**

No agent dispatch. No file writes. No network calls. Pure read + render.

---

## Implementation notes (for the Claude running this command)

- Use the Glob and Read tools directly from the main context. Do NOT spawn an agent — this is a LIGHT utility, dispatching would defeat its purpose.
- Read ≤ 20 files at most for the overview (the glob itself plus a representative frontmatter scan). Use frontmatter-only reads when possible.
- Never invent a command, agent, or skill that isn't on disk. If the glob returns nothing for a category, render "none yet" rather than a placeholder.
- Every number rendered is a count of files found; never cached, never estimated.

---

## Usage

```
/kit:help                             # overview — counts + tier-bucketed command list
/kit:help commands                    # full command table with tier + estimated tokens
/kit:help agents                      # 20 specialists grouped by domain
/kit:help skills                      # 41 skills grouped by cluster
/kit:help enhance                     # resolve to /kit:enhance, print its body
/kit:help frontend-specialist         # resolve to agent, print frontmatter + mission
/kit:help approval-gate               # resolve to skill, print SKILL.md intro
/kit:help kit:debugger                # prefix tolerated; strips and resolves
```
