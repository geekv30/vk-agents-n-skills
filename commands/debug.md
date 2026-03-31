# Debug

Systematically diagnose and fix bugs using root-cause analysis via the SDE2 agent and `superpowers:systematic-debugging`.

## Agents Used

| Agent | Skill | Purpose |
| --- | --- | --- |
| [SDE2](../agents/sde2.md) | superpowers:systematic-debugging + implement-guide | Root-cause analysis and fix |

## Input

- **$ARGUMENTS**: Bug description, error message, failing test, or GitHub issue number

```
Examples:
  /debug "TypeError: Cannot read property 'map' of undefined in SuggestionTable"
  /debug #87                           → Bug from GitHub issue
  /debug ./logs/error.txt              → Error from file
  /debug "Build fails after upgrading React"
```

If no input provided, ask the user to describe the bug or paste the error.

## Flow

```
Parse Input
     │
     ▼
Gather Context (error logs, recent changes, related files)
     │
     ▼
Spawn SDE2 → systematic-debugging → root-cause analysis
     │
     ▼
Present diagnosis to user → WAIT
     │
     ▼
If user approves fix → SDE2 implements fix
     │
     ▼
Verify fix → Report
```

---

## Execution

### Phase 1: Parse Input

```
If #123 found   → gh issue view 123 --json number,title,body,labels,state,url
If file path    → Read file content
Otherwise       → Use the description directly
```

Collect:
- Error message or bug description
- Stack trace (if available)
- Steps to reproduce (if available)
- Expected vs actual behavior

### Phase 2: Gather Context

Before spawning SDE2, collect relevant context:

```bash
# Recent changes that might have caused the bug
git log --oneline -10

# Check for relevant files based on error/description
# (grep for mentioned filenames, components, or error text)
```

### Phase 3: Spawn SDE2 for Diagnosis

**Use the Agent tool. Do NOT debug yourself.**

```
Agent(
  subagent_type: "sde2",
  description: "Debug: {short bug description}",
  prompt: """
  Systematically debug this issue using root-cause analysis.

  ## Bug Report
  {full bug description / error message / issue body}

  ## Context
  - Recent commits: {git log output}
  - Related files: {any files mentioned in error}

  ## Instructions
  1. Invoke `superpowers:systematic-debugging` skill — follow it exactly
  2. DO NOT guess the fix. Form hypotheses, test them, narrow down
  3. Read relevant source files to understand the code
  4. If the bug involves a library, use context7 or WebSearch to check for known issues
  5. Identify the root cause with evidence (specific file, line, condition)
  6. Propose a fix with explanation of WHY it works

  ## Output Format

  ### Root Cause
  {What's actually wrong and why — cite specific file:line}

  ### Evidence
  {How you confirmed this is the root cause, not a symptom}

  ### Proposed Fix
  {Specific code changes with rationale}

  ### Risk Assessment
  {Could this fix break anything else? What to verify after fixing?}
  """
)
```

### Phase 4: Present Diagnosis — WAIT for User

```markdown
## Bug Diagnosis

{SDE2's root cause analysis}

---

**Root cause identified:** {yes/no}
**Confidence:** {high/medium/low}

Should I proceed with the proposed fix?
```

**WAIT for user confirmation.**

### Phase 5: Implement Fix

If user approves, spawn SDE2 again to implement:

```
Agent(
  subagent_type: "sde2",
  description: "Fix: {short bug description}",
  prompt: """
  Implement the approved fix for this bug.

  ## Root Cause
  {diagnosis from Phase 3}

  ## Approved Fix
  {proposed fix from diagnosis}

  ## Instructions
  1. Implement the fix
  2. If tests exist for the affected code, run them
  3. Run typecheck and lint
  4. Commit: `git add -p && git commit -m "fix({scope}): {description}"`
  5. Return summary of changes
  """
)
```

### Phase 6: Verification & Report

Invoke `superpowers:verification-before-completion` to confirm the fix works.

```markdown
## Bug Fixed

### Issue
{original bug description}

### Root Cause
{what was wrong}

### Fix Applied
| File | Change |
|------|--------|
| `path/to/file` | {description} |

### Verification
- TypeScript: {pass/fail}
- Lint: {pass/fail}
- Tests: {pass/fail/N/A}

### Next Steps
- Review changes: `git diff HEAD~1`
- If this was a GitHub issue, consider creating a PR
```

## Error Handling

| Situation | Action |
| --- | --- |
| Can't reproduce the bug | Ask user for more context or reproduction steps |
| Multiple possible root causes | Present all hypotheses with evidence, let user decide |
| Fix requires architectural changes | Present options to user — don't make large changes unilaterally |
| Root cause is in a dependency | Report the finding, suggest workaround or version pin |
