---
description: List kit commands, agents, and skills. Use to discover capabilities. Accepts optional subject: "commands", "agents", "skills", or a specific name.
argument-hint: [commands|agents|skills|<name>]
tier: LIGHT
tier-rationale: Single Read of CATALOG.md — no bash, no glob, no agent dispatch.
estimated-tokens: "<5k"
risk: None — read-only.
---

# /kit:help — Capability index

$ARGUMENTS

---

## Prefix-visibility rule (MANDATORY)

Every primitive rendered carries the `kit:` prefix:
- Commands: `/kit:<name>`
- Agents: `kit:<name>` (the `subagent_type` used to dispatch)
- Skills: `kit:<name>`

Never strip the prefix, even when it looks redundant.

---

## Flow

**Step 1 — Read the catalog (one tool call).**

Read `${CLAUDE_PLUGIN_ROOT}/CATALOG.md`. This file is the authoritative index — no bash, no glob. All counts, descriptions, and groupings come from this single file.

**Step 2 — Classify the argument.**

| `$ARGUMENTS` (trimmed, lowercased) | Mode |
|---|---|
| *empty* | OVERVIEW |
| `commands` / `cmd` | LIST-COMMANDS |
| `agents` / `agent` | LIST-AGENTS |
| `skills` / `skill` | LIST-SKILLS |
| anything else | LOOKUP |

**Step 3 — Render per mode.**

### OVERVIEW (default — no argument)

Render directly from CATALOG.md content. Do not re-glob. Output:

```
# Claude Code Kit  ·  v<version from plugin.json>

17 commands · 20 agents · 42 skills · 16 validation scripts

## Recommended workflow
<copy the workflow block from CATALOG.md verbatim>

## Commands by tier
LIGHT   /kit:budget, /kit:context-budget, /kit:help, /kit:hookify, /kit:instincts,
        /kit:ledger, /kit:preview, /kit:status
MEDIUM  /kit:brainstorm, /kit:debug, /kit:plan, /kit:test (generate mode)
HEAVY   /kit:create, /kit:deploy, /kit:enhance, /kit:orchestrate, /kit:ui-ux-pro-max

MEDIUM/HEAVY commands render an approval gate before dispatching agents.
Pass --yes or -y to any command to bypass the gate (usage still logs).

## Agents by group
<render the four agent groups from CATALOG.md — Architects / Backend-Data-Infra / Frontend-UX / Quality-Ops>
<for each agent: kit:<name> — <what it does one line>>

## Drill deeper
/kit:help commands    — full command table with tier + token ranges
/kit:help agents      — all 20 agents with descriptions
/kit:help skills      — all 42 skills grouped by cluster
/kit:help <name>      — full detail on any one primitive
/kit:ledger weekly    — what you've actually spent this week
/kit:budget [low|medium|ok]  — optional budget hint for the HEAVY gate
```

Read `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json` to get the live version number.

### LIST-COMMANDS

Render the Commands table from CATALOG.md verbatim. Footer: `Run /kit:help <name> for the full body of any command.`

### LIST-AGENTS

Render the Agents section from CATALOG.md verbatim (all four group tables). Footer: `Dispatch via Agent(subagent_type="kit:<name>", prompt=…).`

### LIST-SKILLS

Render the Skills section from CATALOG.md verbatim (all seven clusters). Footer: `Run /kit:help <skill-name> to see that skill's full SKILL.md.`

### LOOKUP `<name>`

Resolution order:
1. Strip optional `kit:` or `/kit:` prefix from the argument.
2. Search `commands/<name>.md`. If found → read it and print the full body with a 2-line header (`## /kit:<name>` + tier/tokens). Return.
3. Search `agents/<name>.md`. If found → read it and print frontmatter + first body section (up to the next `##`). Return.
4. Search `skills/<name>/SKILL.md`. If found → read it and print frontmatter + first H2 section. Footer: `Read the full skill: skills/<name>/SKILL.md`. Return.
5. If nothing matches, case-insensitive grep against the CATALOG.md text for the argument. If there's one likely match, print: `Did you mean /kit:<name>? Run /kit:help <name> to confirm.` If multiple, list top 3.
6. If still nothing: `No kit primitive found matching "<arg>". Run /kit:help to see what's available.`

**Step 4 — Done.**

No agent dispatch. No file writes. No bash. No glob for overview. The LOOKUP step reads at most one additional file (the resolved primitive's file).

---

## Implementation notes

- OVERVIEW and the LIST-* modes pull everything from CATALOG.md. One Read, done.
- LOOKUP reads one more file maximum. Total tool calls: 2 for a lookup, 1 for everything else.
- Never invent a command, agent, or skill not present in CATALOG.md or on disk.
- If CATALOG.md itself is missing (shouldn't happen post-install), fall back to globbing `commands/*.md` and `agents/*.md` — but do NOT run bash; use the Glob tool.

---

## Examples

```
/kit:help                        # overview — workflow + tier buckets + agent roster
/kit:help commands               # full command table
/kit:help agents                 # all 20 specialists grouped
/kit:help skills                 # all 42 skills grouped
/kit:help debug                  # resolve /kit:debug, print its full body
/kit:help debugger               # resolve kit:debugger agent card
/kit:help approval-gate          # resolve skill, print SKILL.md intro
/kit:help kit:frontend-specialist  # prefix tolerated; strips and resolves
```
