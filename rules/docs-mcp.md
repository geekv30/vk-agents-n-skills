# Documentation MCP Server Rules

## MANDATORY: Use Documentation MCP for Library APIs

Two documentation MCP servers are available. **You MUST use them instead of relying on training data** for any library-specific code.

### Available Documentation Sources

| MCP Server | Tools | When to Use |
| ---------- | ----- | ----------- |
| **context7** (plugin) | `resolve-library-id` → `query-docs` | Primary — use for any popular library (React, Tailwind, ShadCN, Vite, etc.). Resolve the library first, then query specific topics. |
| **bob-the-builder-docs-mcp** (local) | Search/fetch tools | Fallback — use when context7 doesn't have the library or for project-specific custom dependencies |

### When to Use

If a task involves a specific framework, library, or API — either explicitly or implicitly — fetch the latest documentation instead of relying on training data. This applies to:

- Writing or modifying code that uses any project dependency
- Answering questions about library APIs, configuration, or patterns
- Debugging issues that may stem from API changes or deprecations
- Suggesting library usage patterns or best practices

### How to Use

1. **Try context7 first**: `resolve-library-id` to find the library, then `query-docs` to fetch relevant docs
2. **Fall back to bob-the-builder-docs-mcp** if context7 doesn't have the library
3. Use the returned documentation to inform your code — not your training data
4. If both are unavailable, explicitly warn: "Documentation MCP servers are not available — this code may use outdated APIs"

### Why This Exists

LLM training data contains deprecated and outdated API patterns. Libraries evolve faster than models are trained. These MCP servers provide real-time, version-specific documentation.

**Do NOT guess APIs from memory. Always verify against indexed documentation.**
