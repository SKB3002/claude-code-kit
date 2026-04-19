# Changelog

All notable changes to Claude Code Kit are documented here. This project follows [Semantic Versioning](https://semver.org/).

## [0.3.0] — 2026-04-19

Minor, non-breaking, additive. All v0.2.1 commands continue to work; six new commands land, agent dispatch becomes deterministic, and the kit gains an approval-first consent layer + usage tracking.

### Added

**Approval-first dispatch (new consent layer)**

- Every `/kit:*` command declares a tier in its frontmatter: `LIGHT` (runs directly), `MEDIUM` (one-line confirm), or `HEAVY` (full gate with planned agents, planned skills, MoSCoW scope, ≥2 lighter alternatives, and a budget line when a budget is set).
- New skill: [`kit:approval-gate`](skills/approval-gate/SKILL.md) + [tiers.md](skills/approval-gate/tiers.md) — the rendering contract, reply parsing, and token-estimation formula.
- `--yes` / `-y` bypass works on all tiers; the usage log still fires.
- Rationale documented in [docs/PLAN-v0.3-user-approval-economy.md](docs/PLAN-v0.3-user-approval-economy.md): the kit targets $20-plan users where weekly rate limits matter, so nothing fans out silently.

**Six new slash commands** (3 LIGHT + 3 targeted adoptions)

- [`/kit:help`](commands/help.md) (LIGHT) — live capability index, globs `commands/`, `agents/`, `skills/*/SKILL.md` at invocation time so new primitives show up automatically.
- [`/kit:budget`](commands/budget.md) (LIGHT) — fully opt-in budget declaration at `~/.kit/budget.json` (home-level default, `--here` for project-local). Absent file = invisible feature.
- [`/kit:ledger`](commands/ledger.md) (LIGHT) — read-only views over `.kit/usage.json`: `weekly`, `by-agent`, `by-skill`, `by-tier`, `command <name>`, `roi`, `verdict <id> <tag>`, `clear`.
- [`/kit:context-budget`](commands/context-budget.md) (LIGHT) — session-scope introspection with GREEN/YELLOW/RED headroom signal and drop-candidate detection. Complements `/kit:ledger` (past spend) by reporting currently loaded context.
- [`/kit:hookify`](commands/hookify.md) (LIGHT) — natural-language → `hooks.json` snippet generator with safety guardrails (Bash PreToolUse warnings, duplicate detection against `hooks.example.json`, `${CLAUDE_PLUGIN_ROOT}` reminders). Never writes to `hooks.json` itself.
- [`/kit:instincts`](commands/instincts.md) (LIGHT) — project-scoped learned preferences at `.kit/instincts.yaml` (git-tracked by default). Synchronous, user-approved promotion — no background capture, no LLM fee on idle.

**Automatic agent dispatch wired into 8 commands**

`/kit:debug`, `/kit:plan`, `/kit:test`, `/kit:brainstorm`, `/kit:enhance`, `/kit:deploy`, `/kit:create`, `/kit:ui-ux-pro-max` now open with explicit `Agent(subagent_type="kit:<name>", prompt=…)` calls instead of prose instructions. Deterministic handoff, not interpretation.

**Usage tracking — `.kit/usage.json`**

Every `/kit:*` run (gated, bypassed, or cancelled) appends one entry with tier declared vs observed, per-agent approximate tokens, skills, files touched, and an open slot for the user's later `verdict useful|wasted|partial` tag. Gitignored. Honesty contract: every token number prefixed `~`, no dollar conversion, no fabricated quota/reset numbers.

**`KIT_HOOK_PROFILE` env var (agent-layer contract)**

`off` / `minimal` / `standard` / `strict` — Claude reads the var at session start and runs the matching validation scripts by protocol (not through Claude Code's hook loader). Documented in [KIT_PROTOCOL.md §6](KIT_PROTOCOL.md) and [hooks/profiles/README.md](hooks/profiles/README.md). No new command — single-variable contract.

**New skills**

- [`kit:approval-gate`](skills/approval-gate/SKILL.md) — gate contract + [tiers.md](skills/approval-gate/tiers.md) (tier taxonomy + token formula).
- [`kit:instincts`](skills/instincts/SKILL.md) — mental model + [schema.md](skills/instincts/schema.md) (YAML reference).

**Renamed skill**

- `kit:brainstorming` → `kit:socratic-gate` to avoid near-collision with `/kit:brainstorm`. Frontmatter notes the former name so external references have a breadcrumb.

### Changed

- All 11 pre-existing commands gained `tier`, `tier-rationale`, `estimated-tokens`, and `risk` frontmatter.
- [`/kit:brainstorm`](commands/brainstorm.md) dispatches `kit:product-manager` explicitly.
- [`/kit:test`](commands/test.md) sub-command branching: `run`/`coverage`/`watch` = LIGHT (no gate); `generate` = MEDIUM gated.
- [README.md](README.md): new "Approval-first by design" section, rendered all primitives with `kit:` prefix for provenance.
- [KIT_PROTOCOL.md](KIT_PROTOCOL.md): new §5 approval-gate rules, new §6 `KIT_HOOK_PROFILE` contract, slash-command table gained a Tier column.
- [.gitignore](.gitignore) adds `.kit/`.

### Fixed

- [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json) — dropped top-level `$schema` and `description` keys that caused strict-validation failures on some Claude Code versions.

### Design stance

From [`agents/product-manager.md`](agents/product-manager.md) + [`agents/product-owner.md`](agents/product-owner.md): **"Build the right thing, on the user's budget, with the user's consent."**

## [0.2.1] — 2026-04-18

### Fixed

- **Plugin failed to load on Claude Code v2.1.x** due to hook-manifest schema mismatch. `hooks.json` and `hooks.example.json` used the legacy flat-event format (`{ "PostToolUse": [...] }`); current Claude Code expects events nested under a top-level `hooks` key (`{ "hooks": { "PostToolUse": [...] } }`). Both files now match the required shape.

## [0.2.0] — 2026-04-18

### ⚠️ Breaking

- **Slash-command namespace renamed from `/claude-code-kit:<name>` to `/kit:<name>`.** The plugin `name` in [.claude-plugin/plugin.json](.claude-plugin/plugin.json) changed from `claude-code-kit` to `kit`. This also changes the `@`-import alias used by host projects: `@claude-code-kit/KIT_PROTOCOL.md` → `@kit/KIT_PROTOCOL.md`.
- Update any host project's `CLAUDE.md` that imports the protocol. The repo, marketplace, and branding remain `claude-code-kit` — only the plugin handle changed.

### Changed

- [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json) plugin entry `name` updated to `kit` to match `plugin.json`.
- Docs ([README.md](README.md), [KIT_PROTOCOL.md](KIT_PROTOCOL.md), [CLAUDE.md](CLAUDE.md), [CONTRIBUTING.md](CONTRIBUTING.md)) updated to use the new `/kit:<name>` prefix and `@kit/` import alias.

## [0.1.0] — 2026-04-17

_First preview release. Structural port of [antigravity-kit](https://github.com/vudovn/antigravity-kit) with Claude Code-native primitives and a stack-aware expansion._

### Added

**Plugin scaffolding**
- `.claude-plugin/plugin.json` manifest
- MIT LICENSE with dual attribution (VUDOVN + Suyash Bhatkar)
- `README.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `.gitignore`

**20 subagents** (`agents/`)
- Architects: `orchestrator`, `project-planner`, `product-owner`, `product-manager`, `code-archaeologist`
- Backend/data/infra: `backend-specialist`, `database-architect`, `devops-engineer`, `security-auditor`, `penetration-tester`, `performance-optimizer`
- Frontend/UX: `frontend-specialist`, `mobile-developer`, `seo-specialist`, `game-developer`
- Quality/ops: `debugger`, `test-engineer`, `qa-automation-engineer`, `documentation-writer`, `explorer-agent`

**40 skills** (`skills/`)
- New for this kit: `fastapi-expert`, `sqlalchemy-expert`, `llm-observability`
- Stack-aware translations for Python/FastAPI/SQLAlchemy across: `lint-and-validate`, `testing-patterns`, `webapp-testing`, `database-design`, `parallel-agents`, `app-builder`, `mcp-builder`
- Full roster mirrors antigravity-kit + the three new Python/LLM skills above

**11 slash commands** (`commands/`)
- `brainstorm`, `create`, `debug`, `deploy`, `enhance`, `orchestrate`, `plan`, `preview`, `status`, `test`, `ui-ux-pro-max`
- All stack-aware — auto-detect Python/Node/Rust/Go

**16 validation scripts** (`skills/<skill>/scripts/`)
- Security: `security_scan.py`
- Lint: `lint_runner.py`, `type_coverage.py`
- Tests: `test_runner.py`, `playwright_runner.py`
- Schema: `schema_validator.py`, `api_validator.py`
- UX: `ux_audit.py`, `accessibility_checker.py`
- React: `react_performance_checker.py`, `convert_rules.py`
- Perf/SEO: `lighthouse_audit.py`, `seo_checker.py`, `geo_checker.py`
- Content: `i18n_checker.py`
- Mobile: `mobile_audit.py`

**Protocol & integration**
- `KIT_PROTOCOL.md` — adapted from Antigravity's `GEMINI.md`, tuned for Claude Code routing, Socratic Gate, and agent/skill enforcement
- `hooks/hooks.json` + `hooks/hooks.example.json` + `hooks/README.md` — opt-in automation, off by default
- `.mcp.json` + `.mcp.example.json` + `mcp-servers.md` — 5 pre-validated MCP servers, opt-in

### Changed from antigravity-kit

- Directory structure flattened: `.agent/agents/` → `agents/`, `.agent/skills/` → `skills/`, workflows → `commands/`
- Paths normalized to `${CLAUDE_PLUGIN_ROOT}/skills/<skill>/scripts/<script>.py`
- Validation scripts co-located with their owning skill (previously centralized under `.agent/scripts/`)
- `/kit:enable-hooks` macro replaced with explicit `hooks.example.json` copy flow
- Skill `react-best-practices` renamed to `nextjs-react-expert` (name now matches the frontmatter)
- Removed orphaned scripts referenced but never shipped: `verify_all.py`, `checklist.py`, `lighthouse_runner.py`, `e2e_runner.py` — mapped to the real equivalents

### Removed

- Antigravity/Windsurf-only rules and path references (`.windsurf/`, `.cursor/`)
- Gemini-specific "mode" mapping (plan/ask/edit) — replaced with Claude Code slash commands

## [Unreleased]

_Next up:_ published plugin-marketplace entry, CI for the validation scripts, a test harness that exercises every agent on a toy repo.
