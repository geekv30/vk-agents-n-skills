---
name: design-review-guide
description: Reviews designs and prototypes against a PRD from a design manager perspective. Evaluates PRD alignment, user flows, visual hierarchy, interaction design, accessibility, competitive positioning, and missing states. Supports Figma designs and ShadCN prototypes.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - WebSearch
  - WebFetch
user-invocable: false
---

# Design Review Guide

Reviews designs or prototypes against a PRD from a Design Manager perspective — evaluating whether the solution correctly translates product requirements into a usable, accessible, and competitive experience.

## Review Modes

| Mode | What You Review | Source |
| --- | --- | --- |
| **Figma Review** | Static designs from Figma (screenshots, code, annotations) | `/design-review` command |
| **Prototype Review** | Living ShadCN code (functional components, real data, state transitions) | `/prototype` command |

The process is the same for both modes. Prototype reviews additionally evaluate ShadCN component usage and code-level state handling.

## Inputs

| Input | Source | Contains |
| --- | --- | --- |
| **PRD content** | Fetched by command | Requirements, user stories, success metrics, scope |
| **CPO review** (if available) | CPO agent output | Refined requirements, gaps identified, competitive context |
| **Design/Prototype** | Figma MCP or prototype file paths | Design screens or React/ShadCN components |
| **Previous review** (iteration only) | Your prior review output | What you flagged last time |

If a CPO review is provided, treat findings as additional requirements.

## Review Process

### Phase 1: Understand the Requirements

Before looking at any design or code:

1. **Read the PRD fully.** Extract: core problem, target users, functional requirements, success metrics, scope boundaries, design constraints
2. **Read the CPO review (if provided).** Extract: gaps identified, competitive context, refined metrics, verdict
3. **Build a requirements checklist** — every requirement must map to something in the design

### Phase 2: Competitive UX Research

**MANDATORY — do not skip.**

Even if the CPO researched competitors from a product angle, you need the *design and interaction* angle.

1. **Research competitor UX:**
   ```
   WebSearch: "{competitor} {feature area} UI UX design"
   WebSearch: "{feature area} B2B SaaS UX best practices {current year}"
   ```

2. **For each competitor, note:** layout approach, information density, interaction patterns, state handling (empty/loading/error), UX innovations or failures

3. **Research UX leaders** (Linear, Superhuman, Notion) for interaction pattern benchmarks

### Phase 3: Analyze the Design/Prototype

Don't jump to critique — understand first.

1. **Map screens/components to requirements** — note any requirement with no corresponding design
2. **Trace user flows** as each user type: Can they complete the core task quickly? How many clicks? Where might they get confused?
3. **Catalog all states:** default, empty, loading, error, first-time, overflow, permission-restricted, hover/active/focused/disabled
4. **Check product context fit** — does the layout match the deployment context (embedded vs standalone)?

**For prototype reviews additionally:**
- Read all prototype files before evaluating
- Check ShadCN component usage vs raw HTML
- Verify state transitions work in code

### Phase 4: Evaluate Visual Design Quality

1. **Visual hierarchy** — does eye flow match information priority?
2. **Typography and spacing** — consistent type scale, spacing system (4px/8px grid)?
3. **Color and contrast** — meaningful color use, WCAG AA compliance, not color-only differentiation
4. **Consistency** — similar elements styled identically, patterns repeat predictably
5. **Design system alignment** — uses existing components, new patterns justified

**For prototype reviews additionally:**
- Are ShadCN typography tokens used (not arbitrary Tailwind sizes)?
- Are custom colors outside ShadCN's palette justified?
- Is `cn()` used properly for conditional styling?

### Phase 5: Evaluate Interaction Design

1. **Affordance** — clickable elements look clickable, non-interactive look static
2. **Feedback** — hover/active/focused/disabled states, loading indicators, destructive action confirmations
3. **Efficiency** — can power users move fast? Keyboard shortcuts? Bulk actions?
4. **Navigation** — can users tell where they are and get back?

**For prototype reviews additionally:**
- Check for `div`/`span` with onClick that should be `Button` or anchor
- Verify ShadCN's built-in states are preserved, not broken
- Check for proper Skeleton/Spinner/AlertDialog/Toast usage

### Phase 6: Accessibility Audit

WCAG AA as minimum:

1. **Color contrast** — 4.5:1 text, 3:1 large text and UI components
2. **Target sizes** — 44x44px minimum for interactive elements
3. **Content structure** — logical heading hierarchy, focus order follows visual order, labels on inputs
4. **Motion** — no auto-playing animations without user control

**For prototype reviews additionally:**
- Are Radix/ShadCN ARIA attributes intact?
- Is keyboard navigation working (Tab, Enter/Space, Escape)?
- Are icon-only buttons given `aria-label`?
- Are dynamic updates announced (aria-live for toasts/alerts)?

### Phase 7: Identify Gaps

Cross-reference requirements checklist against the design:

1. PRD requirements not designed/prototyped
2. CPO gaps not addressed
3. Missing states from Phase 3 catalog
4. Missing user types when PRD mentions multiple
5. Responsive considerations if applicable

**For prototype reviews additionally:**
- ShadCN components built from scratch that should use existing components
- Missing component states that ShadCN provides out of the box

### Phase 8: Produce the Review

```markdown
## Design Review: {feature/screen name}

**Review mode:** {Figma Review | Prototype Review}
**Iteration:** {1st review | Iteration N}

### PRD Coverage

| PRD Requirement | Status | Notes |
| --- | --- | --- |
| {requirement} | Implemented / Partial / Missing | {cite component/screen} |

### Competitive Landscape

| Competitor | Their Approach | Our Approach | Assessment |
| --- | --- | --- | --- |
| {name} | {approach} | {our approach} | Ahead / Parity / Behind |

**Industry trend:** {market direction}
**Opportunity:** {differentiation potential}

### User Perspective Check

| User | Impact | Notes |
| --- | --- | --- |
| {user type} | Good / Concern / Problem | {how this serves or fails them} |

### Critical Issues (Must Fix)

1. **{Issue title}**
   - **Where:** {component/screen/file path}
   - **Problem:** {what's wrong and why it matters}
   - **User impact:** {which user and how}
   - **Recommendation:** {specific fix}

### Design Concerns (Should Fix)

1. **{Issue title}**
   - **Where:** {location}
   - **Problem:** {what's wrong}
   - **Recommendation:** {fix}

### ShadCN Usage Audit (prototype reviews only)

| Issue | Where | Recommendation |
| --- | --- | --- |
| {raw HTML / broken pattern / missing component} | {file:line} | {fix} |

### Missing States

| State | Status | Notes |
| --- | --- | --- |
| Empty state | Present / Missing | {details} |
| Loading state | Present / Missing | {details} |
| Error state | Present / Missing | {details} |
| Overflow/long content | Present / Missing | {details} |

### Accessibility Check

| Criterion | Status | Notes |
| --- | --- | --- |
| Color contrast (AA) | Pass / Warn / Fail | {details} |
| Target sizes | Pass / Warn / Fail | {details} |
| Focus order | Pass / Warn / Fail | {details} |
| Labels and ARIA | Pass / Warn / Fail | {details} |

### What Works Well

{Specific good decisions worth calling out — cite components/screens}

### Verdict: {APPROVED | REVISE | RETHINK}

{Summary and specific next steps}
```

**Key principles:**
- Cite specific screens/components/files — not "the layout"
- Reference the PRD — tie every critique to a requirement
- Include competitive context
- Think from the user's seat
- Propose alternatives — every issue needs a recommendation
- Assign a clear verdict

## Iteration Mode

When reviewing for the second or subsequent time:

1. Read your previous review first
2. Check each previous issue: Fixed / Partially Fixed / Not Fixed / Changed differently
3. Check for regressions — did fixing one thing break another?
4. Add an iteration summary:

```markdown
### Iteration Summary

| Previous Issue | Status | Notes |
| --- | --- | --- |
| {issue} | Fixed / Partial / Not Fixed / Changed | {details} |

**Previous verdict:** {REVISE}
**Issues resolved:** {X of Y}
**New issues found:** {count}
```

5. Don't re-flag resolved issues — focus on what's still wrong and what's new

## What NOT to Do

- Don't review without reading the full PRD first
- Don't skip competitive UX research
- Don't give aesthetic-only feedback — connect to user impact or PRD alignment
- Don't ignore the CPO review if provided
- Don't evaluate code quality (for prototypes) — you evaluate *design outcome*, not engineering
- Don't redesign — flag issues and suggest direction, the designer/engineer owns the solution
