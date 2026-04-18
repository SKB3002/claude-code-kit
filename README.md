# Claude Code Kit

> A comprehensive Claude Code plugin that ships **20 specialist subagents**, **41 skills**, and **14 slash commands** — adapted from [antigravity-kit](https://github.com/vudovn/antigravity-kit) for the Claude Code ecosystem.

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

### Option 1 — Claude Code plugin marketplace (once published)

```bash
claude plugin install claude-code-kit
```

### Option 2 — Clone into your project

```bash
git clone https://github.com/SKB3002/claude-code-kit.git
claude --plugin-dir ./claude-code-kit
```

### Option 3 — Copy into your global plugin directory

```bash
git clone https://github.com/SKB3002/claude-code-kit.git ~/.claude/plugins/claude-code-kit
```

Then activate the kit's routing protocol by adding one line to your project's `CLAUDE.md`:

```
@kit/KIT_PROTOCOL.md
```

That pulls the request classifier, agent routing, and Socratic Gate into every session.

---

## What's inside

| Primitive | Count | Location |
|---|---|---|
| Subagents | 20 | [`agents/`](agents/) |
| Skills | 41 | [`skills/`](skills/) |
| Slash commands | 14 | [`commands/`](commands/) |
| Validation scripts | 16 | `skills/<skill>/scripts/` (co-located with the skill) |
| MCP servers | 5 pre-validated, opt-in | [`.mcp.example.json`](.mcp.example.json) + [`mcp-servers.md`](mcp-servers.md) |
| Hooks | opt-in scaffold | [`hooks/`](hooks/) |

### Slash commands

All commands are namespaced by the plugin — Claude Code renders them as `/kit:<name>`. **Always invoke with the `/kit:` prefix** so it's clear the command is from this kit (and won't collide with built-ins or other plugins):

`/kit:brainstorm` · `/kit:budget` · `/kit:create` · `/kit:debug` · `/kit:deploy` · `/kit:enhance` · `/kit:help` · `/kit:ledger` · `/kit:orchestrate` · `/kit:plan` · `/kit:preview` · `/kit:status` · `/kit:test` · `/kit:ui-ux-pro-max`

Run `/kit:help` in any session to see the full command/agent/skill catalog, rendered live from the plugin's filesystem — counts, tiers, and estimated tokens self-update when primitives are added.

Each command declares a **tier** (LIGHT / MEDIUM / HEAVY) in its frontmatter. MEDIUM and HEAVY commands render an approval gate listing planned agents, skills, estimated token range, and lighter alternatives before they dispatch. LIGHT commands run directly. See the [Approval-first by design](#approval-first-by-design) section below.

### Subagent roster

Dispatch via `Agent(subagent_type="kit:<name>")`. All 20 subagents are namespaced with the `kit:` prefix:

**Architects / leads:** `kit:orchestrator` · `kit:project-planner` · `kit:product-owner` · `kit:product-manager` · `kit:code-archaeologist`
**Backend / data / infra:** `kit:backend-specialist` · `kit:database-architect` · `kit:devops-engineer` · `kit:security-auditor` · `kit:penetration-tester` · `kit:performance-optimizer`
**Frontend / UX:** `kit:frontend-specialist` · `kit:mobile-developer` · `kit:seo-specialist` · `kit:game-developer`
**Quality / ops:** `kit:debugger` · `kit:test-engineer` · `kit:qa-automation-engineer` · `kit:documentation-writer` · `kit:explorer-agent`

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

```bash
# 1) Explore the codebase or frame the problem
/kit:brainstorm auth refactor

# 2) Create a plan (no code yet)
/kit:plan FastAPI rate limiting with Redis

# 3) Orchestrate multiple agents to implement it
/kit:orchestrate implement the plan in docs/PLAN-rate-limit.md

# 4) Run pre-deploy checks
/kit:test
/kit:deploy staging
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
