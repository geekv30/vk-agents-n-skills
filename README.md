# Agents and Skills

This repo stores AI coding workflow configuration:

- `agents/` - agent personas
- `commands/` - slash-command workflows
- `rules/` - auto-loaded rules
- `skills/` - reusable skill instructions
- `.mcp.json` - MCP server config

It is not an app repo. It does not contain a build, backend, or frontend project.

## Getting Started

```bash
git clone https://github.com/geekv30/vk-agents-n-skills.git
cd vk-agents-n-skills
```

Open this folder in your Codex/Claude-style client and use it as the config/project path.

## Setup

No `npm install` is required.

No build step is required.

Node.js is only needed if you want the MCP servers in `.mcp.json` to run locally.

```bash
brew install node
```

Optional: Figma MCP also needs a Figma API token in your client settings.
