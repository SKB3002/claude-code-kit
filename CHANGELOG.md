# Changelog

All notable changes to Claude Code Kit are documented here. This project follows [Semantic Versioning](https://semver.org/).

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
