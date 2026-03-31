# Design Review

Reviews Figma designs against a PRD using a CPO for PRD validation and a Design Manager for design critique. The CPO runs first to validate/refine the PRD, then the Design Manager reviews designs against the refined requirements.

## Agents Used

| Agent | When | Skill |
| --- | --- | --- |
| [CPO](../agents/cpo.md) | Always — validates and refines the PRD before design review begins | [prd-review-guide](../skills/prd-review-guide/SKILL.md) |
| [Design Manager](../agents/design-manager.md) | Always — reviews Figma designs against the PRD (original + CPO refinements) | [design-review-guide](../skills/design-review-guide/SKILL.md) |

## Input

- **$ARGUMENTS**: PRD source + Figma URL (both required)

```
Examples:
  /design-review <notion-url> <figma-url>              → PRD from Notion + Figma designs
  /design-review <confluence-url> <figma-url>           → PRD from Confluence + Figma designs
  /design-review ./specs/feature-prd.md <figma-url>     → PRD from local file + Figma designs
  /design-review #42 <figma-url>                        → PRD from GitHub issue + Figma designs
```

If inputs are missing, ask the user:
- "What's the PRD source? (Notion URL, Confluence URL, file path, or GitHub issue #)"
- "What's the Figma URL for the designs?"

## Flow

```
Parse Inputs
     │
     ▼
Fetch PRD Content (MCP / file / GitHub)
     │
     ▼
Fetch Figma Design Context (Figma MCP)
     │
     ▼
Spawn CPO → validates PRD → returns review + refined requirements
     │
     ▼
Show CPO review to user → WAIT for confirmation
     │
     ▼
Spawn Design Manager → reviews designs against PRD + CPO findings
     │
     ▼
Show Design Review to user
     │
     ▼
Combined Report
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

2. **Figma URL** — must match: `https://*.figma.com/*`

If either is missing, ask the user before proceeding.

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

### Phase 3: Fetch Figma Design Context

Extract `fileKey` and `nodeId` from the Figma URL:
- `figma.com/design/:fileKey/:fileName?node-id=:nodeId` → convert `-` to `:` in nodeId
- `figma.com/design/:fileKey/branch/:branchKey/:fileName` → use branchKey as fileKey

**Fetch design context using Figma MCP:**

```
mcp__claude_ai_figma-mcp-new__get_design_context(fileKey, nodeId)
```

This returns code, screenshots, and contextual hints.

**Also capture a screenshot for visual reference:**

```
mcp__claude_ai_figma-mcp-new__get_screenshot(fileKey, nodeId)
```

Store results as `figma_context` and `figma_screenshot`.

**If Figma fetch fails:** Tell the user. Ask if they can share the Figma file or provide screenshots manually.

### Phase 4: CPO — PRD Review

**Use the Agent tool. Do NOT review the PRD yourself.**

```
Agent(
  subagent_type: "cpo",
  description: "Review PRD for design readiness",
  prompt: """
  Review this PRD from a CPO perspective. Validate the problem, users, metrics, scope, and competitive position before designs are reviewed.

  ## PRD Content
  {prd_content}

  ## Instructions
  1. Read the prd-review-guide skill at `.claude/skills/prd-review-guide/SKILL.md`
  2. Follow the skill's 8-phase review process exactly
  3. MANDATORY: Use WebSearch to research how competitors handle this feature area
  4. Output your review using the exact format defined in your agent persona
  5. Be specific — cite sections of the PRD when flagging issues
  6. Assign a verdict: APPROVED, REFINE, or RETHINK
  """
)
```

Store the CPO's output as `cpo_review`.

### Phase 5: Present CPO Review — WAIT for User

Present the CPO review to the user:

```markdown
## CPO PRD Review Complete

{cpo_review}

---

**Next step:** The Design Manager will now review the Figma designs against this PRD (incorporating the CPO's findings above).

Should I proceed with the design review?
```

**WAIT for user confirmation.**

If the user wants to:
- **Proceed** → Continue to Phase 6
- **Stop here** → End the flow, share the CPO review as the final output
- **Adjust something** → Discuss, then proceed when ready

### Phase 6: Design Manager — Design Review

**Use the Agent tool. Do NOT review the designs yourself.**

```
Agent(
  subagent_type: "design-manager",
  description: "Review Figma designs against PRD",
  prompt: """
  Review these Figma designs against the PRD and CPO findings.

  ## PRD Content
  {prd_content}

  ## CPO Review
  {cpo_review}

  ## Figma Design Context
  {figma_context}

  ## Figma Screenshot
  {figma_screenshot}

  ## Instructions
  1. Read the design-review-guide skill at `.claude/skills/design-review-guide/SKILL.md`
  2. Follow the skill's 8-phase review process exactly
  3. Treat the CPO's findings as additional requirements — any gap the CPO flagged should be checked in the design
  4. MANDATORY: Use WebSearch to research competitor UX for this feature area (design angle, not product angle — the CPO already covered product)
  5. Evaluate from all user perspectives identified in the PRD
  6. Output your review using the exact format defined in your agent persona
  8. Be specific — reference frames, components, and screens by name
  9. Assign a verdict: APPROVED, REVISE, or RETHINK
  """
)
```

Store the Design Manager's output as `design_review`.

### Phase 7: Combined Report

Present the full report to the user:

```markdown
## Design Review Complete: {feature name}

---

### CPO Verdict: {APPROVED | REFINE | RETHINK}

{1-2 sentence summary of CPO findings}

**Key PRD issues:** {count of critical gaps, if any}

<details>
<summary>Full CPO Review</summary>

{cpo_review}

</details>

---

### Design Manager Verdict: {APPROVED | REVISE | RETHINK}

{1-2 sentence summary of design findings}

**Critical issues:** {count}
**Concerns:** {count}
**Missing states:** {count of ❌ items}

<details>
<summary>Full Design Review</summary>

{design_review}

</details>

---

### Combined Assessment

| Area | CPO | Design Manager |
| --- | --- | --- |
| PRD Completeness | {status} | — |
| Competitive Position | {status} | {status} |
| User Coverage | {status} | {status} |
| Design Quality | — | {status} |
| Accessibility | — | {status} |
| Missing States | — | {status} |

### Recommended Next Steps

{Based on both verdicts — what should happen now?}

| CPO Verdict | DM Verdict | Recommendation |
| --- | --- | --- |
| APPROVED | APPROVED | Ready for implementation |
| APPROVED | REVISE | Design iteration needed — PRD is solid, designs need specific fixes |
| APPROVED | RETHINK | Design needs a new direction — schedule design review meeting |
| REFINE | Any | PRD gaps must be addressed first — design review findings are provisional |
| RETHINK | Any | Stop — PRD needs fundamental rework before design continues |
```

---

## When to Ask User

| Situation | Action |
| --- | --- |
| PRD source or Figma URL missing | Ask for the missing input |
| PRD fetch fails (MCP unavailable) | Ask user to paste PRD content |
| Figma fetch fails (MCP unavailable) | Ask user for Figma file access or manual screenshots |
| After CPO review | Always show review and WAIT for confirmation |
| CPO verdict is RETHINK | Warn user that proceeding to design review may not be useful — let them decide |
| User wants to skip CPO step | Allow it — spawn Design Manager directly with original PRD |
