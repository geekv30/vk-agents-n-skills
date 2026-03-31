---
name: ui-prototype-guide
description: Builds a production-grade ShadCN prototype AND self-reviews it against the PRD in a single pass. Combines implementation with design review — build, evaluate, fix, verify. Eliminates multi-agent review loops.
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

# UI Prototype Guide

Builds a production-grade ShadCN prototype from a PRD, then self-reviews it for design quality, and fixes all issues — all in a single pass. Replaces the two-agent loop (builder + reviewer) with a single integrated process.

**Core philosophy: Prototype !== throwaway.** Write code as if it ships tomorrow. Production-quality types, realistic data, proper component composition, full state coverage.

## Inputs

| Input | Source | Contains |
| --- | --- | --- |
| **PRD content** | Fetched by command | Requirements, user stories, success metrics, scope |
| **CPO review** | CPO agent output | Refined requirements, gaps, competitive context |
| **Output directory** | Specified by user or defaulted | Where to write prototype files |

Treat the CPO's "Refined Requirements Summary" as source of truth.

## Process

```
Phase 1: Understand Requirements → Phase 2: Research → Phase 3: Build →
Phase 4: Self-Review → Phase 5: Fix → Phase 6: Verify & Output
(If critical issues remain → repeat Phase 4-5, max 3 iterations)
```

---

## Phase 1: Understand Requirements

Before writing code, internalize what you're building.

1. **Read the PRD fully.** Extract: core problem, target users, functional requirements, success metrics, scope boundaries, design constraints, deployment context (embedded vs standalone)
2. **Read the CPO review.** Extract: gaps identified, competitive context, refined metrics, concerns
3. **Build a requirements checklist** — every item must map to a component in the final prototype
4. **Identify the deployment context** — embedded (constrained viewport, augmenting host app) vs standalone (full-page, own navigation, responsive). This determines your layout strategy, density, and navigation approach.

## Phase 2: Research

### 2.1 Competitive UX Research (MANDATORY)

Research how competitors handle this feature area:

```
WebSearch: "{competitor} {feature area} UI UX design"
WebSearch: "{feature area} B2B SaaS UX best practices {current year}"
```

For each, note: layout approach, density, interaction patterns, state handling, innovations to adopt, failures to avoid.

Also research UX leaders (Linear, Superhuman, Notion) for interaction pattern benchmarks.

**Use findings to inform build decisions — don't just research.**

### 2.2 ShadCN Component Research (MANDATORY)

Before building ANY component, verify what ShadCN offers:

```
WebSearch: "shadcn/ui {component name} component"
```

Map requirements to ShadCN components:

| Need | ShadCN Component | Customization needed? |
| --- | --- | --- |
| {from PRD} | {match} | {yes/no + details} |

### 2.3 Explore Existing Code

If building within an existing project: find similar implementations, identify reusable utilities/hooks/types, note conventions.

## Phase 3: Build

### 3.1 Scaffold

1. Create page/panel layout based on deployment context
2. Set up component file structure — one component per file
3. Define TypeScript interfaces for all data shapes

### 3.2 Build Components

For each PRD requirement:

1. Map to ShadCN component(s)
2. Define typed props interface — no `any`
3. Compose existing ShadCN components — don't reinvent
4. Add realistic mock data — real names, timestamps, email addresses
5. Handle all states:

| State | Implementation |
| --- | --- |
| Empty | Composed empty state with icon + message + CTA |
| Loading | `Skeleton` matching populated layout shape |
| Error | `Alert` with error variant + retry action |
| Populated | Default with realistic data |
| Overflow | 100+ items — pagination, virtualization, or scroll |

### 3.3 Build Rules

| Rule | Detail |
| --- | --- |
| Every UI element uses ShadCN | No raw `<button>`, `<input>`, `<table>` |
| TypeScript strict | No `any`, no untyped props |
| Realistic mock data | No `foo`, `bar`, `test` |
| Component per file | One export with Props interface |
| `cn()` for conditionals | ShadCN's pattern for conditional classes |
| Tailwind only | No inline styles, no CSS modules |
| Context respected | Layout matches deployment context |

## Phase 4: Self-Review (Design Manager Hat)

**Stop thinking like a builder. Think like a Design Manager who just received this prototype.**

### 4.1 PRD Coverage Audit

| PRD Requirement | Status | Component/File |
| --- | --- | --- |
| {requirement} | Implemented / Partial / Missing | {file:line} |

### 4.2 Visual Hierarchy

- Component composition creates clear information priority?
- Primary action most prominent? Secondary actions subordinate?
- Clear reading order? Consistent typography tokens and spacing?
- Density appropriate for deployment context?

### 4.3 Interaction Design

- Clickable elements use proper ShadCN components (Button, Link)?
- No `div`/`span` with onClick that should be Button?
- Hover/active/focused/disabled states preserved?
- Loading states where async operations happen?
- Feedback for destructive actions (AlertDialog, toast)?
- Power user efficiency? Click count for common task?

### 4.4 State Coverage

| State | Present? | Quality |
| --- | --- | --- |
| Empty | Yes/No | {guides user on what to do?} |
| Loading | Yes/No | {matches populated layout shape?} |
| Error | Yes/No | {retry action? helpful message?} |
| Overflow (100+ items) | Yes/No | {pagination, scroll, virtualization?} |

### 4.5 Accessibility

| Criterion | Status |
| --- | --- |
| ShadCN/Radix a11y preserved | {ARIA intact? keyboard nav?} |
| Color contrast (AA) | {custom colors checked?} |
| Target sizes | {44px+ minimum} |
| Focus order | {follows visual order?} |
| Labels and ARIA | {icon-only buttons have aria-label?} |

### 4.6 ShadCN Usage Audit

Scan for: raw `<button>` → `<Button>`, raw `<input>` → `<Input>`, raw `<table>` → `<Table>`, manual `animate-pulse` → `<Skeleton>`, custom styled spans → `<Badge>`.

### 4.7 User Perspective

Walk through as each relevant user type: Can they complete the core task quickly? Where might they get confused?

## Phase 5: Fix Issues

Address every issue from Phase 4, in priority order:

1. Missing PRD requirements
2. State gaps
3. ShadCN violations
4. Interaction issues
5. Accessibility failures
6. Visual hierarchy problems
7. Context violations

Fix structurally, not cosmetically. Re-check states after changes. Don't over-fix passing areas.

## Phase 6: Verify & Output

### Final Verification

Run self-review checklist once more:
- **No critical issues** → produce output
- **Critical issues remain** → repeat Phase 4-5 (max 3 total iterations)

Run technical checks if available:

```bash
npm run typecheck
npm run lint
```

Fix all errors before producing output.

### Output

```markdown
## Prototype Complete: {feature name}

**Deployment context:** {embedded | standalone}
**Self-review iterations:** {1-3}
**Final verdict:** {PASS | PASS WITH NOTES}

### Files Created

| File | Purpose | PRD Requirement |
| --- | --- | --- |
| `path/file.tsx` | {description} | {requirement} |

### PRD Coverage

| PRD Requirement | Status | Component/File |
| --- | --- | --- |
| {requirement} | Implemented / Partial | {file} |

### Self-Review Summary

| Check | Status |
| --- | --- |
| PRD Coverage | Pass / Fixed in iteration N |
| Visual Hierarchy | Pass / Fixed in iteration N |
| Interaction Design | Pass / Fixed in iteration N |
| State Coverage | Pass / Fixed in iteration N |
| Accessibility | Pass / Fixed in iteration N |
| ShadCN Usage | Pass / Fixed in iteration N |
| Data Realism | Pass / Fixed in iteration N |

### Competitive Context

| Competitor | Their Approach | My Approach | Why |
| --- | --- | --- | --- |
| {name} | {approach} | {built} | {rationale} |

### Key Design Decisions

{Notable choices — especially where research or self-review shaped the approach}

### Known Limitations

{Anything not fully addressed and why}
```

## What NOT to Do

- Don't skip research — build quality depends on knowing ShadCN and competitors
- Don't skip self-review — this is the core of the UI Engineer role
- Don't hand-wave — "looks fine" is not a review, check every item
- Don't over-iterate — 3 passes max, then present with notes
- Don't guess ShadCN APIs — WebSearch to verify every component
- Don't use raw HTML when ShadCN has the component
- Don't skip states — empty, loading, error are mandatory
- Don't ignore the CPO review — it contains requirements you must address
