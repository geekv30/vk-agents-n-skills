---
name: implement-guide
description: Implements a single task from an approved plan with production-quality code. Explores existing patterns, researches dependencies, implements, and verifies. Use when implementing individual tasks during feature development.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Edit
  - Write
  - WebSearch
  - WebFetch
user-invocable: false
---

# Implement Guide

Implements a single task from an approved plan. Explore, research, implement, verify, return summary.

## Process

### Step 1: Understand the Task

Read the task description and all context provided:

- What exactly needs to be built
- What's already been completed (from `completed_tasks_log`)
- Architectural notes from the plan
- How your task fits into the overall feature

Do not start coding yet.

### Step 2: Explore Existing Code

Before writing anything:

1. Find similar implementations — how do existing features work?
2. Identify reusable utilities, hooks, and types
3. Note file structure and naming conventions
4. Understand patterns established by completed tasks (check the diff)

### Step 3: Research Dependencies (MANDATORY)

Before writing code, research every third-party dependency you will use:

```
"{library} best practices {current year}"
"{library} {version} documentation"
"{library} common mistakes"
```

Do not guess how a library works. Verify correct usage, configuration options, and common pitfalls.

### Step 4: Implement

1. Follow existing patterns in the codebase exactly
2. Use proper TypeScript types — no `any`
3. Handle errors appropriately
4. Keep code readable and explicit
5. Implement **only this task** — do not begin the next one

### Step 5: Verify

Run after completing:

```bash
npm run typecheck    # TypeScript
npm run lint         # ESLint
```

Fix all errors before returning your summary. Do not claim completion without running these commands and confirming clean output. See [verification-before-completion](#verification-checkpoint) below.

### Verification Checkpoint

Before returning your summary, confirm:

1. You ran `typecheck` and `lint` — not "should pass" but actually ran them
2. You read the output — not assumed success
3. All errors are fixed — zero remaining
4. Your implementation matches the task scope — nothing extra, nothing missing

If you cannot verify (e.g., no build system), state that explicitly in your output.

## Output Format

```markdown
### Task Complete: {task name}

#### Changes Made

| File | Change |
| --- | --- |
| `path/to/file.tsx` | {description} |

#### Key Code

{Most important snippet with file:line reference}

#### Verification

- TypeScript: {No errors | N errors fixed}
- Lint: {No warnings | N warnings fixed}

#### Notes

{Decisions made, patterns established, or anything the next task should know}
```

## Principles

1. **Read before write** — understand existing patterns first
2. **Research before coding** — look up libraries, never guess
3. **Follow loaded rules** — conventions from `.claude/rules/` are the source of truth
4. **One task only** — do not implement beyond your assigned scope
5. **Verify before claiming done** — run checks, read output, confirm clean
6. **Ask when blocked** — surface ambiguity in your summary rather than assuming
