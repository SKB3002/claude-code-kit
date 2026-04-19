# Claude Code Kit

> A comprehensive Claude Code plugin that ships **20 specialist subagents**, **42 skills**, and **17 slash commands** — adapted from [antigravity-kit](https://github.com/vudovn/antigravity-kit) for the Claude Code ecosystem.

Claude Code Kit gives your Claude Code sessions an instant upgrade: domain experts that activate automatically, deep knowledge modules loaded on demand, and workflow commands that orchestrate entire feature builds. Built to augment Claude Code's native features — never to shadow them.

---

## Why this exists

Claude Code ships a powerful agent runtime, but each project tends to reinvent:

- Who the "backend expert" is and what they should know
- Which skills load for a database change vs. a UI change
- Whether to run security audits automatically or on demand

This kit gives you a curated, MIT-licensed starting point. Install it, tweak it, ship it.

---

## Install

### Option 1 — VS Code / Cursor extension (easiest)

1. Open the Claude Code extension sidebar
2. Type `/plugin` → **Manage Plugins** → **Marketplace**
3. Paste: `https://github.com/SKB3002/claude-code-kit`
4. Enable the `kit` plugin → **Reload Window**

> **To update:** go to **Manage Plugins → Marketplace** (not the Plugins section) and click Update next to `claude-code-kit`.

### Option 2 — Claude Code CLI marketplace

```bash
claude plugin install claude-code-kit
```

### Option 3 — Clone and use locally

```bash
git clone https://github.com/SKB3002/claude-code-kit.git
claude --plugin-dir ./claude-code-kit
```

Then activate the routing protocol in your project's `CLAUDE.md`:

```
@kit/KIT_PROTOCOL.md
```

Run `/kit:help` to verify the plugin loaded — it will show all 17 commands, 20 agents, and 42 skills.

---

## Recommended workflow

```
1. /kit:brainstorm <idea>        — explore options before writing any code  (MEDIUM)
2. /kit:plan <feature>           — turn the chosen direction into a plan file  (MEDIUM)
3. /kit:create <app>             — scaffold a greenfield app  (HEAVY)
   or /kit:enhance <change>      — add/update features in an existing app  (HEAVY)
4. /kit:test generate <module>   — generate tests for the new code  (MEDIUM)
   or /kit:test                  — run existing tests  (LIGHT)
5. /kit:deploy staging           — pre-flight + deploy  (HEAVY)
   then /kit:deploy production
```

**Sprinting on a bug?** `/kit:debug` → `/kit:test` → done.

Every MEDIUM/HEAVY command renders an approval gate before dispatching agents — you always see what will run and can cancel or pick a lighter alternative. Pass `--yes` / `-y` to bypass.

---

## Commands (17)

| Command | Tier | ~Tokens | What it does |
|---|---|---|---|
| `/kit:brainstorm <idea>` | MEDIUM | 10k–30k | Explore 3+ options with trade-offs — no code, ideas only |
| `/kit:budget [low\|medium\|ok\|clear]` | LIGHT | <2k | Opt-in budget hint; adds a budget line to the HEAVY gate |
| `/kit:context-budget [verbose]` | LIGHT | <2k | Session headroom signal (GREEN/YELLOW/RED) + drop-candidate detection |
| `/kit:create <what to build>` | HEAVY | 80k–200k | Scaffold a new app — up to 5 specialists from planner through devops |
| `/kit:debug <symptom or error>` | MEDIUM | 15k–40k | Systematic root-cause investigation and fix |
| `/kit:deploy [check\|staging\|production\|rollback]` | HEAVY | 40k–100k | Pre-flight checks, deployment, and post-deploy verification |
| `/kit:enhance <change to make>` | HEAVY | 50k–150k | Add or update features in an existing app |
| `/kit:help [commands\|agents\|skills\|<name>]` | LIGHT | <5k | Full capability index — reads CATALOG.md, no bash |
| `/kit:hookify <nl description>` | LIGHT | <2k | Natural language → hooks.json snippet (never writes the file itself) |
| `/kit:instincts [status\|show\|promote\|clear]` | LIGHT | <2k | Project-scoped learned preferences in `.kit/instincts.yaml` |
| `/kit:ledger [weekly\|by-agent\|by-skill\|roi\|...]` | LIGHT | <3k | Read-only views over `.kit/usage.json` |
| `/kit:orchestrate <task or plan>` | HEAVY | 80k–250k | Coordinate ≥3 agents in a 2-phase plan→approve→implement pipeline |
| `/kit:plan <what to plan>` | MEDIUM | 20k–50k | Generate `docs/PLAN-<slug>.md` — no code, plan file only |
| `/kit:preview [start\|stop\|url]` | LIGHT | <2k | Start/stop the dev server and show the local URL |
| `/kit:status` | LIGHT | <2k | Project state: stack, git, open TODOs, recent changes |
| `/kit:test [generate\|run\|coverage\|watch]` | MEDIUM | 15k–60k | `generate` = MEDIUM (writes tests); other modes = LIGHT |
| `/kit:ui-ux-pro-max <target>` | HEAVY | 60k–180k | Deep UI/UX audit + redesign via frontend-specialist + 3 design skills |

---

## Agents (20)

Dispatch via `Agent(subagent_type="kit:<name>")`. All agents are namespaced `kit:`.

### Architects / Leads

| Agent | What it does |
|---|---|
| `kit:orchestrator` | Multi-agent coordinator — breaks large tasks into parallel slices |
| `kit:project-planner` | Task breakdown, dependency graphs, `docs/PLAN-*.md` output |
| `kit:product-owner` | Requirements, user stories, acceptance criteria, backlog |
| `kit:product-manager` | MoSCoW prioritisation, "build the right thing on the right budget" |
| `kit:code-archaeologist` | Legacy code reading, reverse engineering, modernisation planning |

### Backend / Data / Infra

| Agent | What it does |
|---|---|
| `kit:backend-specialist` | API routes, services, business logic (Node.js, Python/FastAPI, edge) |
| `kit:database-architect` | Schema design, migrations, query optimisation, indexing |
| `kit:devops-engineer` | Deployment, CI/CD, server management, rollbacks — high-risk ops |
| `kit:security-auditor` | OWASP 2025 audits, zero-trust architecture, supply chain security |
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
| `kit:test-engineer` | Test writing, TDD, coverage improvement |
| `kit:qa-automation-engineer` | Playwright, Cypress, E2E pipelines, regression suites |
| `kit:documentation-writer` | READMEs, API docs, changelogs — invoked only on explicit request |
| `kit:explorer-agent` | Deep codebase discovery, architectural analysis, initial audits |

---

## What's inside

| Primitive | Count | Location |
|---|---|---|
| Subagents | 20 | [`agents/`](agents/) |
| Skills | 42 | [`skills/`](skills/) |
| Slash commands | 17 | [`commands/`](commands/) |
| Validation scripts | 16 | `skills/<skill>/scripts/` (co-located with the skill) |
| MCP servers | 5 pre-validated, opt-in | [`.mcp.example.json`](.mcp.example.json) + [`mcp-servers.md`](mcp-servers.md) |
| Hooks | opt-in scaffold | [`hooks/`](hooks/) |

### Highlighted skills

Skills register under the `kit:` namespace (Claude Code's Skill tool resolves them as `kit:<name>`):

- **Cross-stack quality:** `kit:clean-code`, `kit:lint-and-validate`, `kit:testing-patterns`, `kit:tdd-workflow`, `kit:systematic-debugging`, `kit:code-review-checklist`
- **Backend / Python:** `kit:fastapi-expert`, `kit:sqlalchemy-expert`, `kit:python-patterns`, `kit:api-patterns`, `kit:database-design`
- **LLM / AI:** `kit:llm-observability`, `kit:mcp-builder`
- **Frontend:** `kit:frontend-design`, `kit:web-design-guidelines`, `kit:tailwind-patterns`, `kit:nextjs-react-expert`, `kit:mobile-design`
- **Workflow:** `kit:socratic-gate`, `kit:approval-gate`, `kit:plan-writing`, `kit:parallel-agents`, `kit:intelligent-routing`, `kit:behavioral-modes`
- **Security:** `kit:vulnerability-scanner`, `kit:red-team-tactics`
- **Ops:** `kit:deployment-procedures`, `kit:server-management`, `kit:performance-profiling`

Full list: [`skills/`](skills/). Run `/kit:help skills` to list everything live.

> **Convention:** in docs and responses, user-facing references to kit primitives should always carry the `kit:` prefix — so users instantly recognize which capability came from this plugin vs. Claude Code built-ins or other plugins.

---

## Approval-first by design

Most Claude Code Kit users are on the **$20 Pro plan**, where weekly rate limits matter. The kit is built so a `/kit:*` command never silently fans out a fleet of agents on your tokens.

**Tiers**

| Tier | Behaviour | Examples |
|---|---|---|
| **LIGHT** | Runs directly — no gate. | `/kit:status`, `/kit:preview`, `/kit:budget`, `/kit:ledger` |
| **MEDIUM** | One-line preview + `y/n/tweak` prompt. | `/kit:debug`, `/kit:plan`, `/kit:brainstorm`, `/kit:test <target>` |
| **HEAVY** | Full gate — agents, skills, estimated tokens, MoSCoW scope, **alternatives** (always at least one lighter path). | `/kit:create`, `/kit:enhance`, `/kit:deploy`, `/kit:orchestrate`, `/kit:ui-ux-pro-max` |

Power users can pass `--yes` or `-y` as the first argument to bypass any gate. The usage log still fires.

**Usage log — `.kit/usage.json`**

Every run (gated, bypassed, or cancelled) appends one entry to `.kit/usage.json` in the project root (gitignored). Read it via `/kit:ledger`:

- `/kit:ledger weekly` — ISO-week totals by tier, top agents, top skills, cancellations
- `/kit:ledger by-agent` / `by-skill` / `by-tier` — ranked aggregations
- `/kit:ledger command /kit:enhance` — per-command tier drift
- `/kit:ledger roi` — useful/wasted/partial ratios (only from runs **you** tag)
- `/kit:ledger verdict <id> useful|wasted|partial` — tag runs after the fact

**Honesty contract:** every token number is prefixed `~` (approximate, derived from response sizes, not Anthropic billing). No dollar conversion, ever. We never fabricate remaining-quota or reset-timestamp numbers — Claude Code doesn't expose them, so we don't invent them.

**Optional budget file — `~/.kit/budget.json`**

Set once with `/kit:budget low|medium|ok` and it enriches the HEAVY gate with a budget line and a smarter default alternative. If you never run `/kit:budget`, nothing changes — the feature is invisible until you opt in. Fully local, fully deletable (`/kit:budget clear` or `rm ~/.kit/budget.json`).

Design stance from [`agents/product-manager.md`](agents/product-manager.md) + [`agents/product-owner.md`](agents/product-owner.md): **"Build the right thing, on the user's budget, with the user's consent."**

---

## How it augments Claude Code's built-ins

This kit **does not override** Claude Code's built-in skills or subagents. Instead, it layers on top:

- Agents delegate to `Explore` for surveys, `Plan` for step-by-step design, and `general-purpose` for open-ended research
- `security-auditor` can call the built-in `/security-review` and add OWASP-aligned reporting on top
- `clean-code` skill references Claude Code's `/simplify` for canonical cleanup
- `llm-observability` skill assumes the built-in `claude-api` skill is installed

Everything stays familiar — you still use the commands you know.

---

## Hooks — opt-in, never opt-out

`hooks/hooks.json` ships **empty**. The kit never runs scripts on your code without consent.

To wire lint-on-edit, security-on-save, or test-on-stop, copy entries from [`hooks/hooks.example.json`](hooks/hooks.example.json) into `hooks/hooks.json` and restart Claude Code.

Full guide: [hooks/README.md](hooks/README.md).

---

## MCP servers — opt-in

`.mcp.json` ships empty. We pre-validate five servers (filesystem, github, postgres, fetch, sequential-thinking) and document how to enable each one in [mcp-servers.md](mcp-servers.md). Copy from [`.mcp.example.json`](.mcp.example.json) to activate.

---

## Typical workflow

```
# New feature from scratch
/kit:brainstorm caching strategy for LLM gateway   # explore options, no code
/kit:plan Redis token-bucket rate limiting          # outputs docs/PLAN-rate-limit.md
/kit:create FastAPI service with rate limiting      # scaffold the app (HEAVY gate)
/kit:test generate src/middleware/rate_limit.py     # write tests
/kit:deploy staging                                 # pre-flight + deploy

# Improving an existing app
/kit:enhance add dark mode to the dashboard         # HEAVY gate, scoped
/kit:test                                           # run existing suite
/kit:deploy production                              # final deploy

# Debugging
/kit:debug "500 on POST /api/orders under load"     # MEDIUM gate, systematic
/kit:test run                                       # verify fix didn't break anything

# Big cross-cutting change
/kit:orchestrate implement plan in docs/PLAN-auth-refactor.md   # ≥3 agents, 2-phase
```

---

## Contributing

PRs welcome. See [CONTRIBUTING.md](CONTRIBUTING.md). Whether you're adding a subagent, fixing a skill, or proposing a workflow — all contributions are appreciated.

---

## Credits

- Built on [antigravity-kit](https://github.com/vudovn/antigravity-kit) by [@vudovn](https://github.com/vudovn) — the structural blueprint. Licensed MIT.
- Skill content draws from Vercel's React & Web Design Guidelines, OWASP, Core Web Vitals, and the broader open-source community.

---

## License

MIT — see [LICENSE](LICENSE). Use it commercially, personally, or in your side project. Forks, remixes, and derivatives are all welcome.
