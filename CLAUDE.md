# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code **plugin** — not an application. The "product" is prompt content distributed as markdown: 20 subagents, 40 skills, 11 slash commands, plus 16 optional Python validation scripts and opt-in automation scaffolds. There is no build, bundle, or compiled artifact. Editing a file ships the change.

Plugin manifest: [.claude-plugin/plugin.json](.claude-plugin/plugin.json). Authoritative operating rules for host projects: [KIT_PROTOCOL.md](KIT_PROTOCOL.md).

### Heritage

This is a **port** of [vudovn/antigravity-kit](https://github.com/vudovn/antigravity-kit) (originally for Gemini-Antigravity / Windsurf / Cursor) into Claude Code's plugin format. [KIT_PROTOCOL.md](KIT_PROTOCOL.md) is the adapted `GEMINI.md`. The [LICENSE](LICENSE) carries dual MIT attribution (VUDOVN + Suyash Bhatkar) — keep both when editing it. The initial commit was merged onto an empty GitHub-generated repo via `--allow-unrelated-histories -X ours` so the curated `LICENSE` and `README.md` won; do not rebase history before `cf6906d`.

The port added stack-aware Python/FastAPI/SQLAlchemy/Alembic coverage to existing skills (`lint-and-validate`, `testing-patterns`, `webapp-testing`, `database-design`, `app-builder`, `mcp-builder`, `parallel-agents`) **without removing** the Node/Next.js content. Three new skills shipped alongside: `fastapi-expert`, `sqlalchemy-expert`, `llm-observability`. Keep that "additive, not replacing" discipline when editing existing skills.

## Local "run" / test

There is no test suite. To exercise the kit inside Claude Code:

```bash
claude --plugin-dir ./claude-code-kit
```

Validation scripts (listed in [KIT_PROTOCOL.md §5](KIT_PROTOCOL.md)) are **manual** — they are never auto-invoked by the kit. Run one directly with:

```bash
python skills/<skill>/scripts/<script>.py
```

Script conventions (all 16 already comply — match them for new ones):

- Shebang `#!/usr/bin/env python3`
- Stdlib-only where possible
- Accept `sys.argv[1]` with `.` (cwd) as default target
- Self-locate with `Path(__file__).parent` — **never** read `${CLAUDE_PLUGIN_ROOT}` from inside a script. That env var is only for referencing scripts **from outside** (hook configs, agent instructions): `python ${CLAUDE_PLUGIN_ROOT}/skills/<skill>/scripts/<script>.py`.
- No hardcoded `.windsurf/`, `.cursor/`, or `.agent/` paths — that cleanup was done during the port from antigravity-kit. Don't reintroduce them.

## Architecture

Four top-level primitive folders, each with a rigid convention. Do not reorganize them — `KIT_PROTOCOL.md` and the plugin loader both address files by path.

- **[agents/](agents/)** — flat `.md` files, one per subagent. YAML frontmatter is load-bearing: `name`, `description`, `tools`, `model`, `skills`. The `skills:` list drives the *modular skill loading* flow described below.
- **[skills/](skills/)** — one folder per skill, entry point is `SKILL.md`. Optional sub-reference files (e.g., `rest.md`, `graphql.md`) are pulled in selectively. Validation scripts for a skill live **inside** that skill's folder at `skills/<skill>/scripts/` — they are co-located on purpose so the skill owns its own tooling.
- **[commands/](commands/)** — flat `.md` files. Claude Code auto-namespaces them as `/kit:<filename>` (the `kit` prefix comes from the `name` field in [plugin.json](.claude-plugin/plugin.json) — the repo/marketplace is still `claude-code-kit`, only the plugin handle is `kit`). Do **not** prefix the filename yourself.
- **[hooks/](hooks/)** — ships `hooks.json` intentionally empty. Real examples live in `hooks.example.json`; users copy entries in to opt in. Never auto-enable anything here.

Cross-cutting:

- **[KIT_PROTOCOL.md](KIT_PROTOCOL.md)** is the plugin's "runtime." Host projects import it via `@kit/KIT_PROTOCOL.md` in their own `CLAUDE.md` (the alias matches the plugin name, not the repo name). Any change to agent routing, tier rules, validation script inventory, or slash-command mapping belongs here — and must stay in sync with the underlying files it references.
- **[.mcp.json](.mcp.json)** ships empty by design; examples live in [.mcp.example.json](.mcp.example.json), documented in [mcp-servers.md](mcp-servers.md). Same opt-in discipline as hooks.

### The modular skill-loading flow

This is the one non-obvious architectural contract across the kit (from [KIT_PROTOCOL.md §1](KIT_PROTOCOL.md)):

```
Agent invoked → read agent .md → check frontmatter `skills:` → open each SKILL.md → pull only the sections needed
```

When adding or modifying an agent, the `skills:` array in its frontmatter is how its domain knowledge gets loaded. Skills referenced there must exist as `skills/<name>/SKILL.md`. When adding a skill, its `SKILL.md` should read as an index first so callers can load only what they need.

### Rule priority

P0 ([KIT_PROTOCOL.md](KIT_PROTOCOL.md)) > P1 (agent `.md`) > P2 (skill `SKILL.md`). When editing content, keep that hierarchy intact — narrower files should not contradict broader ones.

## Conventions that matter for edits

- **Descriptions tell the model *when* to use the thing, not *what* it does.** This applies to agent, skill, and slash-command frontmatter — it is how Claude Code's auto-routing picks them.
- **Subagents are personas with a philosophy**, not feature lists. See [agents/orchestrator.md](agents/orchestrator.md) for the established voice.
- **Slash commands are not "modes."** The kit deliberately has no mode switcher — intent is expressed by command choice ([KIT_PROTOCOL.md §5 slash-command mapping](KIT_PROTOCOL.md)).
- **Kit augments, never shadows, Claude Code built-ins.** Agents delegate surveys to the built-in `Explore` agent, step-by-step design to `Plan`, and open-ended research to `general-purpose`. The `security-auditor` agent wraps `/security-review`; `clean-code` skill references `/simplify`; `llm-observability` assumes the built-in `claude-api` skill. When editing, preserve these delegations rather than reimplementing them.
- **When adding a validation script**, also register it in the script table in [KIT_PROTOCOL.md §5](KIT_PROTOCOL.md) and (if it should run automatically for someone) in [hooks/hooks.example.json](hooks/hooks.example.json) — never in `hooks.json`.
- **Dependent-file awareness:** agents, skills, commands, KIT_PROTOCOL.md, and README.md cross-reference each other by relative path. When renaming or moving a file, grep the repo for the old path and update every reference.
- **Slash-command files use `$ARGUMENTS`** as the user-input placeholder and require `description:` + `argument-hint:` frontmatter. They are stack-aware — `/test` branches pytest vs. `npm test`, `/preview` branches uvicorn vs. `npm run dev`, `/deploy` branches on detected stack. Preserve that branching when editing.
- **`/plan` emits `docs/PLAN-{slug}.md`** — slug is derived from user input, ≤30 chars, lowercase, hyphens.
- **`/orchestrate` requires ≥3 agents and 2-phase execution** (Plan → user approval → Implement) and enforces a *Context Passing MANDATORY* rule: each parallel agent must be handed the full relevant context in its prompt because parallel agents cannot see each other's work. Don't edit out that section.

## Gotchas (landmines from the port)

- **Don't re-prefix slash-command filenames with `kit-`.** The old antigravity pattern is obsolete — Claude Code namespaces via `/kit:<name>` automatically (driven by `plugin.json` `name`).
- **`react-best-practices` was renamed to `nextjs-react-expert`.** Skill folder and frontmatter both match the new name. Don't reintroduce the old name in references.
- **`.agent/scripts/verify_all.py` and `checklist.py` do not exist here.** The upstream protocol referenced them, but they were never shipped. Removed during the port — don't re-add without an actual implementation.
- **JSON has no comments.** `hooks.json` and `.mcp.json` are strict JSON and must stay clean. The `_doc` key convention is allowed **only** in the `*.example.json` files, never the live ones.
- **Windows LF→CRLF warnings from git are expected** and not a bug.
- **`ui-ux-pro-max` no longer uses a proprietary design search script** (the upstream did). It now delegates to `frontend-design`, `web-design-guidelines`, and `tailwind-patterns` via `frontend-specialist`. Keep that delegation.



