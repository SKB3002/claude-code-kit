# Contributing to Claude Code Kit

Thanks for your interest in improving the kit. All contributions are welcome — bug fixes, new agents, new skills, better workflows, or just sharper descriptions.

## Ways to contribute

- **Fix a typo or improve a description** — go straight to a PR
- **Add a new skill** — open an issue first describing the domain, then PR
- **Add a new subagent** — open an issue with the intended role and which skills it would load
- **Propose a new slash command** — open an issue with the intended workflow
- **Port a script** — PR welcome; include a brief note on what it validates

## Development workflow

1. Fork the repo
2. Create a feature branch: `git checkout -b feat/your-change`
3. Test locally against Claude Code:
   ```bash
   claude --plugin-dir ./claude-code-kit
   ```
4. Commit with a descriptive message
5. Open a PR against `main`

## Conventions

- **Subagents** live in `agents/` as flat `.md` files with YAML frontmatter (`name`, `description`, `tools`, `model`, `skills`).
- **Skills** live in `skills/<skill-name>/SKILL.md` with optional sub-reference files (e.g., `rest.md`, `graphql.md`).
- **Slash commands** live in `commands/` as flat `.md` files. Claude Code namespaces them as `/kit:<name>` — do not prefix filenames.
- **Validation scripts** live in `skills/<skill-name>/scripts/` co-located with the skill that owns them. Keep them dependency-light; prefer stdlib.
- **Hooks** (optional, off by default) live in `hooks/hooks.json`. Ship examples in `hooks/hooks.example.json` and document them in `hooks/README.md`. Do not auto-enable.
- **MCP servers** (optional, off by default) follow the same pattern: `.mcp.json` stays empty, examples live in `.mcp.example.json`, documented in `mcp-servers.md`.

## Style

- Descriptions: tell the model **when** to use the thing, not **what it does**.
- Agents: frame as a persona with a clear philosophy, not a feature list.
- Skills: lead with a content map if the skill has sub-files.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
