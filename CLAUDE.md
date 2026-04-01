# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A **Claude Code configuration repository** — agents, commands, skills, and rules that govern how Claude Code operates in development workflows. No application code lives here; prototype apps targeted by commands like `/prototype` live in sibling directories (e.g., `try-claude/prototype/`).

## New Machine Setup

```bash
# 1. Clone
git clone https://github.com/geekv30/vk-agents-n-skills.git
cd vk-agents-n-skills

# 2. Install Node.js (required for MCP servers — context7, playwright)
brew install node   # or https://nodejs.org

# 3. Install plugins (skills are bundled, but installing plugins keeps them up-to-date)
claude plugin install superpowers
claude plugin install frontend-design
claude plugin install context7
claude plugin install playwright
claude plugin install figma

# 4. Optional: machine-local permission overrides (gitignored)
cp settings.local.json.example settings.local.json
```

**MCP servers** (context7, playwright, figma) are declared in `.mcp.json` and auto-start via `npx` — no separate install needed beyond Node.js. Figma requires a one-time API token in Claude Code's Figma integration settings.

**Skills** (superpowers, frontend-design) are bundled in `skills/` as a fallback. If the plugins are also installed (recommended), the plugin versions take precedence and stay up-to-date automatically.

> **Conflict note**: If both the bundled skills and user-scope plugins are active simultaneously, Claude Code may see duplicate skill definitions. To avoid this, either uninstall the user-scope plugins (`claude plugin uninstall superpowers`) and rely solely on the bundled copies, or remove the bundled dirs (`skills/superpowers/`, `skills/frontend-design/`) and rely solely on the plugins.

## Repository Structure

```
.
├── agents/            # Agent persona definitions (6 agents)
├── commands/          # Multi-agent orchestration commands (9 commands)
├── rules/             # Auto-loaded rules injected into every conversation (4 rules)
├── skills/            # SKILL.md methodology files
│   ├── superpowers/   # Bundled superpowers skills (brainstorming, TDD, debugging, etc.)
│   ├── frontend-design/ # Bundled frontend-design skill
│   └── ...            # Project-specific skills (11)
├── .mcp.json          # MCP server configs: context7, playwright, figma
├── settings.local.json # Machine-local overrides (gitignored)
└── CLAUDE.md          # This file
```

## File Format Conventions

All agents, commands, and skills are Markdown files with YAML frontmatter.

**Agents** (`agents/*.md`):
```yaml
---
name: agent-name
description: When to use this agent
tools: Read, Grep, Glob, Bash, Edit, Write, WebSearch, WebFetch
model: opus
permissionMode: bypassPermissions
memory: project  # persistent cross-session learning
skills:
  - skill-name
---
# Agent body: persona, mindset, base rules, skill-specific rules
```

**Commands** (`commands/*.md`): Plain Markdown (no frontmatter). Define input format, agent orchestration flow, and output expectations. Commands are invoked as `/command-name` in Claude Code.

**Skills** (`skills/{name}/SKILL.md`):
```yaml
---
name: skill-name
description: What this skill does
allowed-tools: [Read, Grep, Glob, Bash, Edit, Write, WebSearch, WebFetch]
user-invocable: false  # true if callable directly via /skill-name
---
# Skill body: purpose, methodology, checklists
```

## Which Command Should I Use?

```
What are you trying to do?
│
├── Build something new
│   ├── UI/prototype from a PRD ──────────── /prototype
│   │                                         (add --thorough for Design Manager review)
│   └── Feature / backend / full-stack ────── /implement
│                                             (add --review for per-task architect review)
│
├── Review or fix code
│   ├── Review a PR ──────────────────────── /pr-review
│   │                                        (add --follow-up for re-review after changes)
│   ├── Fix PR review feedback ───────────── /pr-fix
│   └── Debug a bug ─────────────────────── /debug
│
├── Write or run tests ───────────────────── /test
│
├── Review designs (Figma + PRD) ─────────── /design-review
│
├── Update documentation ─────────────────── /update-docs
│
└── Check dependency health ──────────────── /analyze-deps
```

## Architecture

### Agents (6)

| Agent | Role | Key Skills |
|-------|------|------------|
| **sde2** | Implementation, code quality, correctness, docs, deps | implement-guide, pr-fix-guide, pr-review-guide, update-docs-guide, analyze-deps-guide |
| **principal-architect** | System design, scalability, architecture | design-plan, pr-review-guide |
| **tester** | Test strategy and implementation | implement-test-guide |
| **cpo** | PRD validation, product strategy, competitive analysis | prd-review-guide |
| **design-manager** | UI/UX design review against product requirements | design-review-guide |
| **ui-engineer** | Builds AND self-reviews ShadCN prototypes | ui-prototype-guide, shadcn, figma skills, frontend-design |

### Commands (9)

| Command | Flow |
|---------|------|
| `/implement` | Brainstorm -> Architect plans -> SDE2 implements (`--review` for per-task architect review loop) |
| `/pr-review` | Architect + SDE2 review in parallel (`--follow-up` for re-review after changes) |
| `/pr-fix` | Systematic debugging -> SDE2 fixes PR review feedback |
| `/debug` | Root-cause analysis -> SDE2 diagnoses and fixes bugs |
| `/test` | Tester agent drives test creation |
| `/prototype` | CPO refines PRD -> UI Engineer builds + self-reviews (`--thorough` for Design Manager review loop) |
| `/design-review` | CPO validates PRD -> Design Manager reviews Figma designs |
| `/update-docs` | SDE2 updates documentation |
| `/analyze-deps` | SDE2 analyzes dependency updates |

### Project Skills (11)

| Skill | Used By | Purpose |
|-------|---------|---------|
| implement-guide | sde2 | Feature implementation methodology |
| pr-review-guide | sde2, principal-architect | PR review (includes follow-up mode) |
| pr-fix-guide | sde2 | Fix PR review feedback |
| design-plan | principal-architect | Architecture planning |
| implement-test-guide | tester | Test implementation |
| prd-review-guide | cpo | PRD validation |
| design-review-guide | design-manager | Design/prototype review (includes iteration mode) |
| ui-prototype-guide | ui-engineer | Prototype build + self-review process |
| update-docs-guide | sde2 | Documentation updates |
| analyze-deps-guide | sde2 | Dependency analysis |
| shadcn | ui-engineer | ShadCN component patterns (symlinked) |

### Rules (auto-loaded every conversation)

| Rule | Scope |
|------|-------|
| **workflow.md** | Phase-based execution: Plan -> Execute ONE phase -> Summarize -> WAIT for confirmation |
| **docs-mcp.md** | Mandatory use of context7/bob-the-builder-docs-mcp for library APIs |
| **github.md** | Conventional commits, PR templates, branch naming, `gh` CLI patterns |
| **prototype-conventions.md** | React 19 + Vite 7 + ShadCN v4 + Tailwind CSS v4 stack conventions |

### Bundled vs. External Dependencies

| Dependency | How it ships | Notes |
|-----------|--------------|-------|
| **superpowers** skills | Bundled in `skills/superpowers/` | No install needed |
| **frontend-design** skill | Bundled in `skills/frontend-design/` | No install needed |
| **context7** MCP | `.mcp.json` → `npx @upstash/context7-mcp` | Needs Node.js |
| **playwright** MCP | `.mcp.json` → `npx @playwright/mcp@latest` | Needs Node.js |
| **figma** MCP | `.mcp.json` → `https://mcp.figma.com/mcp` | Needs Figma API token |

## Key Workflow Rules

1. **Phase-based execution**: Plan (TodoWrite) -> Execute one phase -> Summarize -> WAIT for user confirmation -> Repeat
2. **Never execute multiple phases without stopping for review**
3. **Brainstorm before creative work**: Invoke `superpowers:brainstorming` before building features or components
4. **Verify before completion**: Invoke `superpowers:verification-before-completion` before claiming work is done
5. **Research-first**: Use context7 (primary) or bob-the-builder-docs-mcp (fallback) for library APIs — never guess from training data
6. **GitHub conventions**: Conventional commits (`feat(scope): description`), `Closes #N` in PR bodies, `Co-Authored-By` footer

## Modifying This Repository

When adding or editing agents, commands, or skills:

- Follow the YAML frontmatter format shown above — Claude Code parses these fields
- Agent `skills:` values must match directory names under `skills/`
- Commands reference agents via relative paths (e.g., `../agents/sde2.md`)
- After significant changes, check if `rules/*.md` files or this CLAUDE.md need updates
