# Agents and Skills

Portable Claude Code configuration — clone this repo on any machine to instantly pick up agents, commands, skills, and workflow rules.

Not an app repo. No build step, no backend, no frontend.

## Quick Start

```bash
git clone https://github.com/geekv30/vk-agents-n-skills.git
cd vk-agents-n-skills

# MCP servers need Node.js
brew install node

# Install plugins (keeps skills up-to-date; bundled copies work as fallback)
claude plugin install superpowers
claude plugin install frontend-design
claude plugin install context7
claude plugin install playwright
claude plugin install figma

# Optional: machine-local permission overrides
cp settings.local.json.example settings.local.json
```

Open this folder as your project path in Claude Code — everything loads automatically.

## What's in the Repo

| Path | Purpose |
|------|---------|
| `agents/` | 6 agent personas (sde2, architect, tester, cpo, design-manager, ui-engineer) |
| `commands/` | 9 slash commands (`/implement`, `/prototype`, `/debug`, `/pr-review`, etc.) |
| `rules/` | 4 auto-loaded rules (workflow, docs-mcp, github, prototype-conventions) |
| `skills/` | 11 project skills + bundled superpowers + frontend-design |
| `.mcp.json` | MCP server configs for context7, playwright, figma |
| `settings.local.json` | Machine-local overrides — gitignored, copy from `.example` |

## Dependencies

| Dependency | Ships as | Needs |
|-----------|----------|-------|
| superpowers skills | Bundled in `skills/superpowers/` | Nothing |
| frontend-design skill | Bundled in `skills/frontend-design/` | Nothing |
| context7 MCP | `.mcp.json` → `npx @upstash/context7-mcp` | Node.js |
| playwright MCP | `.mcp.json` → `npx @playwright/mcp@latest` | Node.js |
| figma MCP | `.mcp.json` → `https://mcp.figma.com/mcp` | Figma API token |

> **Note**: If the plugins are also installed at user scope, you may see duplicate skill definitions. Either uninstall the user-scope plugins, or remove `skills/superpowers/` and `skills/frontend-design/` from the repo and rely on the plugins alone.
