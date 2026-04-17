# Claude Code Kit

> A comprehensive Claude Code plugin that ships **20 specialist subagents**, **40 skills**, and **11 slash commands** — adapted from [antigravity-kit](https://github.com/vudovn/antigravity-kit) for the Claude Code ecosystem.

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
@claude-code-kit/KIT_PROTOCOL.md
```

That pulls the request classifier, agent routing, and Socratic Gate into every session.

---

## What's inside

| Primitive | Count | Location |
|---|---|---|
| Subagents | 20 | [`agents/`](agents/) |
| Skills | 40 | [`skills/`](skills/) |
| Slash commands | 11 | [`commands/`](commands/) |
| Validation scripts | 16 | `skills/<skill>/scripts/` (co-located with the skill) |
| MCP servers | 5 pre-validated, opt-in | [`.mcp.example.json`](.mcp.example.json) + [`mcp-servers.md`](mcp-servers.md) |
| Hooks | opt-in scaffold | [`hooks/`](hooks/) |

### Slash commands

All commands are namespaced by the plugin — Claude Code renders them as `/claude-code-kit:<name>` (or use your own alias):

`brainstorm` · `create` · `debug` · `deploy` · `enhance` · `orchestrate` · `plan` · `preview` · `status` · `test` · `ui-ux-pro-max`

### Subagent roster

**Architects / leads:** `orchestrator` · `project-planner` · `product-owner` · `product-manager` · `code-archaeologist`
**Backend / data / infra:** `backend-specialist` · `database-architect` · `devops-engineer` · `security-auditor` · `penetration-tester` · `performance-optimizer`
**Frontend / UX:** `frontend-specialist` · `mobile-developer` · `seo-specialist` · `game-developer`
**Quality / ops:** `debugger` · `test-engineer` · `qa-automation-engineer` · `documentation-writer` · `explorer-agent`

### Highlighted skills

- **Cross-stack quality:** `clean-code`, `lint-and-validate`, `testing-patterns`, `tdd-workflow`, `systematic-debugging`, `code-review-checklist`
- **Backend / Python:** `fastapi-expert`, `sqlalchemy-expert`, `python-patterns`, `api-patterns`, `database-design`
- **LLM / AI:** `llm-observability`, `mcp-builder`
- **Frontend:** `frontend-design`, `web-design-guidelines`, `tailwind-patterns`, `nextjs-react-expert`, `mobile-design`
- **Workflow:** `brainstorming`, `plan-writing`, `parallel-agents`, `intelligent-routing`, `behavioral-modes`
- **Security:** `vulnerability-scanner`, `red-team-tactics`
- **Ops:** `deployment-procedures`, `server-management`, `performance-profiling`

Full list: [`skills/`](skills/)

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
/claude-code-kit:brainstorm auth refactor

# 2) Create a plan (no code yet)
/claude-code-kit:plan FastAPI rate limiting with Redis

# 3) Orchestrate multiple agents to implement it
/claude-code-kit:orchestrate implement the plan in docs/PLAN-rate-limit.md

# 4) Run pre-deploy checks
/claude-code-kit:test
/claude-code-kit:deploy staging
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
