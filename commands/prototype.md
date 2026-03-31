# Prototype

Transforms a PRD into a production-grade ShadCN prototype. Default mode uses the UI Engineer (single-agent build + self-review). Use `--thorough` for a multi-agent loop with independent Design Manager review.

## Agents Used

| Agent | Default Mode | Thorough Mode |
| --- | --- | --- |
| [CPO](../agents/cpo.md) | Always — validates and refines PRD | Always |
| [UI Engineer](../agents/ui-engineer.md) | Builds + self-reviews prototype | Builds prototype, iterates on DM feedback |
| [Design Manager](../agents/design-manager.md) | Not used | Reviews prototype, gives iteration feedback |

## Input

- **$ARGUMENTS**: PRD source (required) + optional output directory + optional flags

```
Examples:
  /prototype <notion-url>                          -> PRD from Notion (default mode)
  /prototype ./specs/feature-prd.md                -> PRD from local file
  /prototype #42                                   -> PRD from GitHub issue
  /prototype ./specs/feature-prd.md ./prototype    -> PRD + custom output directory
  /prototype #42 --thorough                        -> Thorough mode with Design Manager review
  /prototype #42 -t                                -> Short flag for thorough
```

If the PRD source is missing, ask the user:
- "What's the PRD source? (Notion URL, Confluence URL, file path, or GitHub issue #)"

If no output directory is specified, default to `./prototype`.

## Mode Comparison

| | Default | Thorough (`--thorough`) |
| --- | --- | --- |
| Build agent | UI Engineer | UI Engineer |
| Review agent | UI Engineer (self-review) | Design Manager (independent) |
| Agent handoffs | 0 (single agent) | 4 (build -> review -> iterate -> re-review) |
| User wait points | 3 | 5 |
| Quality control | Built-in self-review loop (up to 3 iterations) | External design critique |
| Best for | Speed with comparable quality | When independent external critique matters |

## Flow

### Default Mode

```
Parse Inputs -> Fetch PRD -> Spawn CPO -> WAIT
     -> Spawn UI Engineer (build + self-review) -> Playwright Verify -> Report
```

### Thorough Mode

```
Parse Inputs -> Fetch PRD -> Spawn CPO -> WAIT
     -> Spawn UI Engineer (build) -> WAIT
     -> Spawn Design Manager (review) -> WAIT
     -> Spawn UI Engineer (iterate on feedback) -> WAIT
     -> Spawn Design Manager (final review) -> Playwright Verify -> Report
```

---

## Execution

### Phase 1: Parse Inputs

Parse `$ARGUMENTS` to identify:

1. **PRD source** — one of:

   | Pattern | Type |
   | --- | --- |
   | `https://*.notion.so/*` or `https://*.notion.site/*` | Notion URL |
   | `https://*.atlassian.net/*` | Confluence URL |
   | `*.md` or `*.txt` file path | Local file |
   | `#123` | GitHub issue |
   | Anything else | Ask user to clarify |

2. **Output directory** (optional) — any path-like argument that isn't the PRD source. Default: `./prototype`

3. **Mode flag** — `--thorough` or `-t`. Default: standard mode.

### Phase 2: Fetch PRD Content

Based on PRD source type:

**Notion URL:**
```
Use Notion MCP tools to fetch the page content.
Extract: title, full body content, any linked databases or sub-pages.
```

**Confluence URL:**
```
Use Atlassian MCP tools to fetch the page content.
Extract: title, full body content, any child pages.
```

**Local file:**
```
Read the file using the Read tool.
```

**GitHub issue:**
```bash
gh issue view {number} --json number,title,body,labels,state,url
```

Store the full PRD content as `prd_content`.

**If PRD fetch fails:** Tell the user and ask them to paste the PRD content directly.

### Phase 3: CPO — PRD Review

**Use the Agent tool. Do NOT review the PRD yourself.**

```
Agent(
  subagent_type: "cpo",
  description: "Review PRD for prototype readiness",
  prompt: """
  Review this PRD from a CPO perspective. Validate the problem, users, metrics, scope, and competitive position before prototyping begins.

  ## PRD Content
  {prd_content}

  ## Instructions
  1. Read the prd-review-guide skill at `.claude/skills/prd-review-guide/SKILL.md`
  2. Follow the skill's review process exactly
  3. MANDATORY: Use WebSearch to research how competitors handle this feature area
  4. Output your review using the exact format defined in your agent persona
  5. Be specific — cite sections of the PRD when flagging issues
  6. Assign a verdict: APPROVED, REFINE, or RETHINK
  7. IMPORTANT: Produce a "Refined Requirements Summary" at the end — a consolidated list of all requirements (original PRD + your additions/refinements). The UI Engineer will use this as the source of truth.
  """
)
```

Store the CPO's output as `cpo_review`.

### Phase 4: Present CPO Review — WAIT for User

```markdown
## CPO PRD Review Complete

{cpo_review}

---

**CPO Verdict:** {APPROVED | REFINE | RETHINK}

**Mode:** {Default — UI Engineer will build + self-review | Thorough — UI Engineer builds, Design Manager reviews independently}

Should I proceed with prototyping?
```

**WAIT for user confirmation.**

- **Proceed** -> Continue
- **Stop here** -> End the flow
- **Adjust something** -> Discuss, then proceed

**If CPO verdict is RETHINK:** Warn that prototyping against a fundamentally flawed PRD may waste effort. Let the user decide.

---

## Default Mode: Phase 5 — UI Engineer Build + Self-Review

**Use the Agent tool. Do NOT build the prototype yourself.**

```
Agent(
  subagent_type: "ui-engineer",
  description: "Build and self-review ShadCN prototype",
  prompt: """
  Build a production-grade ShadCN prototype AND self-review it against the PRD — all in a single pass.

  ## PRD Content
  {prd_content}

  ## CPO Review (Refined Requirements)
  {cpo_review}

  ## Output Directory
  {output_directory}

  ## Instructions
  1. Read the ui-prototype-guide skill at `.claude/skills/ui-prototype-guide/SKILL.md` — follow the process exactly
  2. The CPO's "Refined Requirements Summary" is your source of truth
  3. MANDATORY: Research competitors for this feature area using WebSearch
  4. MANDATORY: Research ShadCN component availability using WebSearch before building
  5. Build the prototype using React + TypeScript + ShadCN + Tailwind
  6. Cover ALL requirements from the refined PRD
  7. Include all states: empty, loading, error, populated, overflow
  8. Use realistic mock data — real-looking names, timestamps, emails
  9. Every UI element MUST use ShadCN components
  10. CRITICAL: After building, run your full self-review. Be ruthless. Up to 3 iterations.
  11. Output using the exact format defined in the skill
  """
)
```

Store as `prototype_result`. Skip to Phase 9 (Playwright Verification).

---

## Thorough Mode: Phase 5T — UI Engineer Build

**Use the Agent tool. Do NOT build the prototype yourself.**

```
Agent(
  subagent_type: "ui-engineer",
  description: "Build ShadCN prototype from PRD",
  prompt: """
  Build a production-grade ShadCN prototype based on this refined PRD.

  ## PRD Content
  {prd_content}

  ## CPO Review (Refined Requirements)
  {cpo_review}

  ## Output Directory
  {output_directory}

  ## Instructions
  1. Read the ui-prototype-guide skill at `.claude/skills/ui-prototype-guide/SKILL.md`
  2. The CPO's "Refined Requirements Summary" is your source of truth
  3. MANDATORY: Use WebSearch to verify ShadCN component availability before building
  4. Build the prototype using React + TypeScript + ShadCN + Tailwind
  5. Cover ALL requirements from the refined PRD
  6. Include all states: empty, loading, error, populated, overflow
  7. Use realistic mock data — real-looking names, timestamps, emails
  8. Every UI element MUST use ShadCN components
  9. After building, list all files created and which PRD requirements each file addresses

  ## Output Format
  - List of all files created with brief descriptions
  - Requirements coverage table: PRD requirement -> component/file
  - Any requirements you couldn't prototype and why
  """
)
```

Store as `prototype_summary`.

**Present to user and WAIT.**

### Phase 6T: Design Manager — Review Prototype

```
Agent(
  subagent_type: "design-manager",
  description: "Review prototype against PRD",
  prompt: """
  Review this ShadCN prototype against the PRD and CPO findings.

  ## PRD Content
  {prd_content}

  ## CPO Review
  {cpo_review}

  ## Prototype Summary
  {prototype_summary}

  ## Prototype Files
  The prototype is located at: {output_directory}
  Read ALL prototype files to understand the full implementation.

  ## Instructions
  1. Read the design-review-guide skill at `.claude/skills/design-review-guide/SKILL.md`
  2. Follow the skill's review process — use "Prototype Review" mode
  3. Read EVERY file in the prototype directory before critiquing
  4. Treat CPO findings as additional requirements
  5. MANDATORY: Use WebSearch to research competitor UX for this feature area
  6. Evaluate from all user perspectives identified in the PRD
  7. Audit ShadCN usage — flag raw HTML where ShadCN components should be used
  8. Be specific — reference component names and file paths
  9. Assign a verdict: APPROVED, REVISE, or RETHINK
  10. For each issue, write actionable feedback the UI Engineer can directly implement
  """
)
```

Store as `dm_review_1`.

**Present to user and WAIT.**

If DM verdict is **APPROVED**, skip to Phase 9 (Playwright Verification).

### Phase 7T: UI Engineer — Iterate on Feedback

```
Agent(
  subagent_type: "ui-engineer",
  description: "Iterate prototype on DM feedback",
  prompt: """
  Iterate on the prototype based on the Design Manager's feedback.

  ## Original PRD Content
  {prd_content}

  ## CPO Review
  {cpo_review}

  ## Design Manager Review
  {dm_review_1}

  ## Prototype Location
  {output_directory}

  ## Instructions
  1. Read the Design Manager's review completely
  2. Read ALL existing prototype files to understand current state
  3. Address Critical Issues FIRST, then Design Concerns
  4. Address ShadCN Usage Audit findings
  5. Address Missing States flagged by the DM
  6. MANDATORY: Use WebSearch to verify ShadCN usage for any new components
  7. Do NOT change things the DM praised — preserve what works
  8. After iterating, list all changes and which DM feedback item each addresses

  ## Output Format
  - Issue resolution table: DM issue -> change made -> files modified
  - List of all files modified or added
  - Any DM feedback you couldn't address and why
  """
)
```

Store as `iteration_summary`.

**Present to user and WAIT.**

### Phase 8T: Design Manager — Final Review

```
Agent(
  subagent_type: "design-manager",
  description: "Final prototype review after iteration",
  prompt: """
  This is your SECOND review — the UI Engineer has iterated based on your previous feedback.

  ## PRD Content
  {prd_content}

  ## CPO Review
  {cpo_review}

  ## Your Previous Review
  {dm_review_1}

  ## UI Engineer's Iteration Summary
  {iteration_summary}

  ## Prototype Files
  The updated prototype is at: {output_directory}
  Read ALL prototype files to see the current state.

  ## Instructions
  1. Read the design-review-guide skill — follow "Iteration Mode"
  2. Read EVERY file in the prototype directory
  3. Check each previous issue: Fixed, Partial, Not Fixed, Changed
  4. Check for regressions
  5. Don't re-flag resolved issues — focus on remaining and new problems
  6. Assign a final verdict: APPROVED, REVISE, or RETHINK
  """
)
```

Store as `dm_review_final`.

---

## Phase 9: Visual Verification with Playwright (Optional)

If the Playwright MCP plugin is available:

1. Start the dev server:
   ```bash
   cd {output_directory} && npm run dev &
   ```

2. Use Playwright MCP to navigate and screenshot:
   ```
   browser_navigate -> http://localhost:5173
   browser_take_screenshot -> capture the main view
   ```

3. Analyze for visual issues: broken layout, missing content, rendering problems.

4. Report issues alongside the combined report.

5. Stop the dev server after verification.

**Skip** if Playwright MCP is unavailable or no dev server is configured.

---

## Phase 10: Combined Report

### Default Mode Report

```markdown
## Prototype Complete: {feature name}

**Prototype location:** {output_directory}
**Build method:** UI Engineer (single-agent build + self-review)

---

### CPO Verdict: {APPROVED | REFINE | RETHINK}

{1-2 sentence summary}

<details>
<summary>Full CPO Review</summary>

{cpo_review}

</details>

---

### UI Engineer Result

{prototype_result}

---

### Prototype Summary

| Metric | Value |
| --- | --- |
| Total files | {count} |
| PRD requirements covered | {X of Y} |
| Self-review iterations | {1-3} |
| Self-review verdict | {PASS / PASS WITH NOTES} |
| ShadCN components used | {list} |
| States implemented | {list} |

---

### Recommended Next Steps

| CPO Verdict | UI Engineer Verdict | Recommendation |
| --- | --- | --- |
| APPROVED | PASS | Ready for production implementation |
| APPROVED | PASS WITH NOTES | Solid with minor notes. Review, then proceed |
| REFINE | Any | Address PRD gaps before production |
| RETHINK | Any | PRD needs fundamental rework |
```

### Thorough Mode Report

```markdown
## Prototype Complete: {feature name}

**Prototype location:** {output_directory}
**Build method:** Thorough (UI Engineer + Design Manager review loop)

---

### CPO Verdict: {APPROVED | REFINE | RETHINK}

{1-2 sentence summary}

<details>
<summary>Full CPO Review</summary>

{cpo_review}

</details>

---

### Design Manager Final Verdict: {APPROVED | REVISE | RETHINK}

{1-2 sentence summary}

**Issues from first review:** {total}
**Resolved after iteration:** {count}
**Remaining:** {count}

<details>
<summary>First Design Review</summary>

{dm_review_1}

</details>

<details>
<summary>Final Design Review</summary>

{dm_review_final}

</details>

---

### Prototype Summary

| Metric | Value |
| --- | --- |
| Total files | {count} |
| PRD requirements covered | {X of Y} |
| ShadCN components used | {list} |
| States implemented | {list} |

---

### Recommended Next Steps

| CPO Verdict | DM Final Verdict | Recommendation |
| --- | --- | --- |
| APPROVED | APPROVED | Ready for production implementation |
| APPROVED | REVISE | Targeted fixes needed before production |
| APPROVED | RETHINK | Needs different direction — schedule design sync |
| REFINE | Any | Address PRD gaps first |
| RETHINK | Any | PRD needs fundamental rework |
```

---

## When to Ask User

| Situation | Action |
| --- | --- |
| PRD source missing | Ask for it |
| PRD fetch fails | Ask user to paste PRD content |
| After CPO review | Always WAIT |
| CPO verdict is RETHINK | Warn, let user decide |
| After prototype built (thorough) | Always WAIT |
| After DM review (thorough) | Always WAIT |
| DM gives APPROVED on first review | Skip iteration |
| User wants to skip CPO step | Allow — spawn UI Engineer directly |

## Configuration

| Setting | Default | Override |
| --- | --- | --- |
| Mode | Default (single-agent) | `--thorough` / `-t` |
| Max self-review iterations | 3 | Internal to UI Engineer |
| Max DM iteration rounds | 1 | Hardcoded |
| Output directory | `./prototype` | Second argument |
| CPO step | Enabled | User can skip |
| ShadCN-only | Enforced | Non-negotiable |
