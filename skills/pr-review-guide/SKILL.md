---
name: pr-review-guide
description: Reviews pull requests for code quality, architecture, and project conventions. Supports both full reviews and follow-up re-reviews after changes are pushed. Use when reviewing PRs or verifying fixes after review feedback.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash(git:*, gh:*)
  - WebSearch
  - WebFetch
user-invocable: false
---

# PR Review

Reviews pull requests to ensure code quality, architectural soundness, and adherence to project conventions.

## Review Process

1. **Understand the context**
   - Read PR description and linked GitHub issues
   - Understand what problem is being solved
   - Check the scope of changes

2. **Research third-party dependencies (MANDATORY)**
   - Identify ALL third-party libraries in the changed code
   - Use WebSearch to find latest documentation and common mistakes
   - Verify correct usage and version-specific behavior
   - See [External Technology Research](#external-technology-research) below

3. **Analyze the changes**
   - Review each changed file thoroughly — don't skim the diff
   - Consider how changes fit into the existing system
   - Compare implementation against researched best practices

4. **Apply your lens**
   - Use your agent's focus areas (architecture vs implementation)
   - Check against your agent's checklist items

5. **Document findings**
   - Use the [Output Format](#output-format) below
   - Be specific with file:line references
   - Distinguish blocking vs minor issues

## External Technology Research

**MANDATORY for any PR using third-party dependencies.**

### What to Search For

1. **Correct usage** — Is the API being used as intended?
2. **Configuration options** — Are there settings that should be customized?
3. **Common mistakes** — What do people typically get wrong?
4. **Version-specific behavior** — Check `package.json` versions against docs
5. **Performance considerations** — Any known gotchas?

### Research Output

Include in your review:

```markdown
### External Technology Research

| Technology | Findings | Impact on PR |
| --- | --- | --- |
| {name} | {key finding from docs} | {recommendation} |
```

## Common Checklist

- [ ] No exposed secrets or API keys
- [ ] No console.log or debugging code left in
- [ ] No commented-out code without explanation
- [ ] Imports organized and minimal
- [ ] Follows existing patterns in codebase
- [ ] Changes are within scope of PR description
- [ ] Third-party dependencies used correctly (verified via research)

## Agent-Specific Focus

### Principal Architect Focus

- System design and patterns
- Scalability implications
- Data model correctness
- API contract clarity
- Security boundaries
- Extensibility

### SDE2 Focus

- Type safety
- Error handling
- Code structure and readability
- Naming conventions
- Edge cases
- Testability

## Output Format

```markdown
## {Agent Name} Review — Verdict: {VERDICT}

### Issues

| # | Severity | File | Issue | Suggestion |
| --- | --- | --- | --- | --- |
| 1 | Blocking | `file.ts:42` | {concise description} | {only if fix is non-obvious} |
| 2 | Minor | `file.ts:15` | {concise description} | {only if fix is non-obvious} |

### Architectural Concerns

{Only for genuine design-level issues. Use: **{Title}** (`file.ts:line`) — 1-2 sentences. Omit section if none.}

### Research Flags

{Only if dependency research found misuse. Omit section if none.}

| Technology | Issue | Recommendation |
| --- | --- | --- |
| {name} | {what's wrong} | {what to do} |
```

**Rules:**
- Omit any section entirely if it has no content
- No "Observations", "What's done well", or praise
- No summary status table
- `Suggestion` column: only populate when the fix isn't obvious

## Returning Findings

**Do NOT post the review yourself.** Return structured findings for the caller:

```
REVIEW_BODY:
{your review body}

INLINE_COMMENTS:
[
  {"path": "src/button.tsx", "line": 42, "body": "**Issue:** ..."}
]

VERDICT: APPROVE | COMMENT | REQUEST_CHANGES
```

### Finding Line Numbers

Use `gh pr diff {pr_number}` — line numbers must be from the **new file** (right side of diff) and on lines that appear in the diff.

## Verdict Options

| Agent | Verdicts |
| --- | --- |
| **Principal Architect** | `APPROVED`, `NEEDS DISCUSSION`, `CHANGES REQUIRED` |
| **SDE2** | `APPROVED`, `MINOR CHANGES`, `CHANGES REQUIRED` |

| Condition | Verdict |
| --- | --- |
| No issues found | `APPROVED` |
| Only minor style/preference issues | `MINOR CHANGES` or `NEEDS DISCUSSION` |
| Bugs, missing error handling, architectural issues | `CHANGES REQUIRED` |

---

## Follow-up Mode

Use this mode when re-reviewing a PR after the author pushed changes in response to initial feedback.

### Follow-up Process

1. **Gather previous issues** — list all blocking and minor issues from the prior review
2. **Identify changed files** — get commits since last review, list modified files
3. **Research dependencies (if applicable)** — if fixes involve third-party libraries, verify against latest docs
4. **Verify previous issues** — for each, check if addressed using status indicators below
5. **Review new changes only** — don't re-review unchanged code; focus on files modified since last review
6. **Apply follow-up verdict logic**

### Status Indicators

| Status | Meaning |
| --- | --- |
| Fixed | Issue has been addressed |
| Still Present | Issue remains unchanged |
| Partially Fixed | Attempted but incomplete |
| Changed | Code changed, needs re-evaluation |

### Follow-up Output Format

```markdown
## {Agent Name} Follow-up — Verdict: {VERDICT}

### Unresolved Issues

| # | Original Issue | File | Status | Notes |
| --- | --- | --- | --- | --- |
| 1 | {description} | `file.ts:42` | Still Present | {what's still wrong} |
| 2 | {description} | `file.ts:15` | Partial | {what's missing} |

### New Issues

| # | Severity | File | Issue | Suggestion |
| --- | --- | --- | --- | --- |
| 1 | Blocking | `file.ts:10` | {description} | {only if non-obvious} |
```

**Rules:**
- `Unresolved Issues`: only rows for issues Still Present or Partial. No Fixed rows.
- If all resolved, replace table with: _All previous issues resolved._
- `New Issues`: omit section entirely if none
- No praise or "what's done well"

### Follow-up Verdict Guidelines

**SDE2:**

| Condition | Verdict |
| --- | --- |
| All blocking fixed, no new issues | `APPROVED` |
| Previous fixed, only minor new issues | `MINOR CHANGES` |
| Blocking issues remain OR new blocking issues | `CHANGES REQUIRED` |

**Principal Architect:**

| Condition | Verdict |
| --- | --- |
| Previous architectural concerns addressed | `APPROVED` |
| Minor questions remain, non-blocking | `NEEDS DISCUSSION` |
| Significant architectural issues remain | `CHANGES REQUIRED` |
