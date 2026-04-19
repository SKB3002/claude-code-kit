---
description: Pre-built capability index for /kit:help. One Read call — no glob, no bash. Update this file when adding a primitive.
---

# Claude Code Kit — Capability Catalog (v0.3.0)

> This file is read by `/kit:help` instead of globbing the filesystem.
> **When you add a command, agent, or skill: update this file and README.md.**

---

## Recommended workflow

```
1. /kit:brainstorm <idea>       — explore options before committing (MEDIUM)
2. /kit:plan <feature>          — turn chosen direction into a plan file (MEDIUM)
3. /kit:create <app>            — scaffold greenfield app (HEAVY)
   or /kit:enhance <change>     — add/update features in existing app (HEAVY)
4. /kit:test generate <module>  — generate tests for the new code (MEDIUM)
   or /kit:test                 — run existing tests (LIGHT)
5. /kit:deploy staging          — pre-flight + deploy (HEAVY)
   then /kit:deploy production
```

Sprinting on a bug? `/kit:debug` → `/kit:test` → done.
Need a second opinion? `/kit:brainstorm` → `/kit:plan` before any code.

---

## Commands (17)

| Command | Tier | ~Tokens | What it does |
|---|---|---|---|
| `/kit:brainstorm <idea>` | MEDIUM | 10k–30k | Explore 3+ options with trade-offs via product-manager — no code, ideas only |
| `/kit:budget [low\|medium\|ok\|clear]` | LIGHT | <2k | Opt-in budget declaration; enriches HEAVY gate with budget line + recommendation |
| `/kit:context-budget [verbose]` | LIGHT | <2k | Session headroom signal (GREEN/YELLOW/RED) + drop-candidate detection |
| `/kit:create <what to build>` | HEAVY | 80k–200k | Scaffold a new application — 4–5 specialists from planner through devops |
| `/kit:debug <symptom or error>` | MEDIUM | 15k–40k | Systematic root-cause investigation via kit:debugger + fix |
| `/kit:deploy [check\|staging\|production\|rollback]` | HEAVY | 40k–100k | Pre-flight checks, deployment, and post-deploy verification |
| `/kit:enhance <change to make>` | HEAVY | 50k–150k | Add or update features in an existing app — minimum agent set, scoped |
| `/kit:help [commands\|agents\|skills\|<name>]` | LIGHT | <5k | This command — reads CATALOG.md, no bash, no glob |
| `/kit:hookify <nl description>` | LIGHT | <2k | Convert natural language to a hooks.json snippet — never writes the file itself |
| `/kit:instincts [status\|show\|promote\|clear]` | LIGHT | <2k | Project-scoped learned preferences stored in .kit/instincts.yaml |
| `/kit:ledger [weekly\|by-agent\|by-skill\|roi\|...]` | LIGHT | <3k | Read-only aggregations over .kit/usage.json |
| `/kit:orchestrate <task or plan>` | HEAVY | 80k–250k | Coordinate ≥3 agents in Plan→approve→Implement pipeline |
| `/kit:plan <what to plan>` | MEDIUM | 20k–50k | Generate docs/PLAN-<slug>.md — no code, plan file only |
| `/kit:preview [start\|stop\|url]` | LIGHT | <2k | Start/stop the dev server and show the local URL |
| `/kit:status` | LIGHT | <2k | Project state: stack, git, open TODOs, recent changes |
| `/kit:test [generate\|run\|coverage\|watch]` | MEDIUM | 15k–60k | `generate` = MEDIUM (writes tests); `run`/`coverage`/`watch` = LIGHT (no gate) |
| `/kit:ui-ux-pro-max <target>` | HEAVY | 60k–180k | Deep UI/UX audit + redesign — frontend-specialist + 3 design skills |

Pass `--yes` or `-y` to any command to bypass the gate (MEDIUM/HEAVY). Usage still logs.

---

## Agents (20)

### Architects / Leads

| Agent | What it does |
|---|---|
| `kit:orchestrator` | Multi-agent coordinator — breaks large tasks into parallel slices |
| `kit:project-planner` | Task breakdown, dependency graphs, docs/PLAN-*.md output |
| `kit:product-owner` | Requirements, user stories, acceptance criteria, backlog management |
| `kit:product-manager` | MoSCoW prioritisation, "build the right thing on the right budget" |
| `kit:code-archaeologist` | Legacy code reading, reverse engineering, modernisation planning |

### Backend / Data / Infra

| Agent | What it does |
|---|---|
| `kit:backend-specialist` | API routes, services, business logic (Node.js, Python/FastAPI, edge) |
| `kit:database-architect` | Schema design, migrations, query optimisation, indexing |
| `kit:devops-engineer` | Deployment, CI/CD, server management, rollbacks — high-risk ops |
| `kit:security-auditor` | OWASP 2025 audits, zero-trust, supply chain security |
| `kit:penetration-tester` | Offensive security, red team, exploit simulations (CTF / engagement) |
| `kit:performance-optimizer` | Profiling, Core Web Vitals, bundle size, runtime bottlenecks |

### Frontend / UX

| Agent | What it does |
|---|---|
| `kit:frontend-specialist` | React, Next.js, Vue, Svelte — components, state, responsive design |
| `kit:mobile-developer` | React Native, Flutter — cross-platform mobile apps and native features |
| `kit:seo-specialist` | SEO audits, Core Web Vitals, E-E-A-T, AI search (GEO) visibility |
| `kit:game-developer` | Unity, Godot, Phaser, Three.js — mechanics, multiplayer, 2D/3D |

### Quality / Ops

| Agent | What it does |
|---|---|
| `kit:debugger` | Systematic root-cause analysis — the specialist for hard bugs |
| `kit:test-engineer` | Test writing, TDD, coverage improvement — unit through integration |
| `kit:qa-automation-engineer` | Playwright, Cypress, E2E pipelines, regression suites |
| `kit:documentation-writer` | READMEs, API docs, changelogs — invoked only on explicit request |
| `kit:explorer-agent` | Deep codebase discovery, architectural analysis, initial audits |

---

## Skills (42)

### Cross-stack quality (7)
`kit:clean-code` · `kit:lint-and-validate` · `kit:testing-patterns` · `kit:tdd-workflow` · `kit:systematic-debugging` · `kit:code-review-checklist` · `kit:webapp-testing`

### Backend / Python (10)
`kit:fastapi-expert` · `kit:sqlalchemy-expert` · `kit:python-patterns` · `kit:api-patterns` · `kit:database-design` · `kit:app-builder` · `kit:nodejs-best-practices` · `kit:bash-linux` · `kit:powershell-windows` · `kit:rust-pro`

### LLM / AI (2)
`kit:llm-observability` · `kit:mcp-builder`

### Frontend (9)
`kit:frontend-design` · `kit:web-design-guidelines` · `kit:tailwind-patterns` · `kit:nextjs-react-expert` · `kit:mobile-design` · `kit:geo-fundamentals` · `kit:seo-fundamentals` · `kit:i18n-localization` · `kit:game-development`

### Workflow (9)
`kit:socratic-gate` · `kit:approval-gate` · `kit:plan-writing` · `kit:parallel-agents` · `kit:intelligent-routing` · `kit:behavioral-modes` · `kit:instincts` · `kit:architecture` · `kit:documentation-templates`

### Security (2)
`kit:vulnerability-scanner` · `kit:red-team-tactics`

### Ops (3)
`kit:deployment-procedures` · `kit:server-management` · `kit:performance-profiling`
