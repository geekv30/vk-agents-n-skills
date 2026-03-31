---
name: prd-review-guide
description: Review and refine PRDs from a product strategy, competitive, and business outcome perspective. Validates problem clarity, user coverage, success metrics, scope, and competitive positioning.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - WebSearch
  - WebFetch
user-invocable: false
---

# PRD Review Guide

## Purpose

Review a PRD (Product Requirements Document) from a CPO perspective — validating that the right problem is being solved, for the right users, with the right scope, measured by the right metrics, and positioned correctly against the competitive landscape.

## When to Use

- Before design begins on a new feature
- When the design-review command spawns you to validate a PRD before passing it to the design manager
- When a PM asks for PRD feedback
- When a feature direction needs product leadership input

## Inputs

You will receive the PRD content from the command that spawned you. The PRD may come from:
- Notion (fetched via MCP by the command)
- Atlassian/Confluence (fetched via MCP by the command)
- A local markdown file (read by the command)
- Pasted text in conversation

## Review Process

### Phase 1: Parse the PRD Structure

Read the full PRD and extract these elements. If any are missing, that's already a finding.

| Element | What to Extract | If Missing |
| --- | --- | --- |
| Problem statement | What user problem is being solved? | Critical gap — flag immediately |
| Evidence | Data, research, user quotes backing the problem | Flag: "Problem lacks evidence" |
| Target users | Which specific user types are addressed? | Flag: "No clear target user" |
| User stories/jobs | What can users do after this ships? | Flag: "No user outcomes defined" |
| Success metrics | How will we measure this worked? | Critical gap — can't evaluate without this |
| Scope (in) | What's being built in this release? | Flag: "Scope boundaries unclear" |
| Scope (out) | What's explicitly deferred? | Flag: "No out-of-scope section" |
| Cross-channel | How does this work across email, chat, KB, voice? | Flag if feature touches user-facing flows |
| Gmail constraints | How does this work within Gmail? | Flag if feature has UI components |
| Edge cases | Error states, permissions, scale, migration | Flag what's missing |
| Rollout plan | How do users discover and adopt this? | Flag: "No adoption strategy" |

### Phase 2: Competitive Research

**MANDATORY — do not skip this phase.**

1. Identify the feature area from the PRD
2. Research how direct competitors handle the same problem:

   ```
   WebSearch: "Intercom {feature area} 2025 2026"
   WebSearch: "Zendesk {feature area} features capabilities"
   WebSearch: "Front {feature area} how it works"
   WebSearch: "Freshdesk {feature area} 2025 2026"
   WebSearch: "Pylon {feature area} customer service"
   WebSearch: "{feature area} B2B customer service trends {current year}"
   ```

3. For each competitor, document:
   - Do they have this feature? Since when?
   - How does their implementation work?
   - What's their UX approach?
   - What do users say about it (reviews, complaints)?

4. Synthesize:
   - Is this table stakes or differentiation for Hiver?
   - What patterns are emerging as industry standard?
   - Where has a competitor failed that Hiver can learn from?
   - What's the opportunity to leapfrog?

### Phase 3: Problem Validation

Evaluate the problem statement critically:

1. **Is the problem real?**
   - Is there evidence (user interviews, support tickets, usage data, churn signals)?
   - Or is this an assumed problem based on competitor features or stakeholder requests?

2. **Is the problem sized correctly?**
   - Too broad: "Improve agent productivity" — needs narrowing
   - Too narrow: "Add a button to the toolbar" — this is a solution, not a problem
   - Right size: "Agents spend 3+ minutes per ticket on manual tagging, causing SLA breaches during peak hours"

3. **Is this the right problem to solve now?**
   - What's the urgency? (competitive pressure, user churn, scale blocker?)
   - What's the opportunity cost? (what doesn't get built if we build this?)

### Phase 4: User Coverage Analysis

For each user type relevant to this feature:

**Internal Users:**
- **Support Agent:** Does the PRD address their daily workflow impact? Speed, clicks, cognitive load?
- **Support Manager:** Does the PRD address oversight, reporting, configuration needs?
- **Admin:** Does the PRD address setup, rollout, permissions?

**External Users (if applicable):**
- **End Customer (portal):** Self-serve impact? Ticket deflection?
- **End Customer (chat):** Response time? Bot-to-human handoff?
- **End Customer (KB):** Content discoverability? Search quality?

Flag any user type that's affected but not addressed in the PRD.

### Phase 5: Success Metrics Evaluation

For each proposed metric, evaluate:

| Quality | What It Looks Like | Example |
| --- | --- | --- |
| ✅ Strong | Specific, measurable, time-bound, tied to outcome | "70% of agents use auto-tagging within 30 days; avg tagging time drops from 45s to 5s" |
| ⚠️ Weak | Measurable but not outcome-oriented | "Feature adoption rate" (how much? by when? so what?) |
| ❌ Missing/Bad | Vague, unmeasurable, or output-focused | "Users can tag tickets" (that's a feature description, not a metric) |

If metrics are missing or weak, propose specific alternatives.

### Phase 6: Scope & Feasibility Check

1. **v1 boundaries:**
   - Is the minimum viable scope clear?
   - Are deferred items explicitly listed?
   - Is there a logical progression from v1 → future iterations?

2. **Cross-channel:**
   - If the feature touches user-facing flows, does it address all relevant channels?
   - If single-channel, is the rationale for deferring other channels clear?

3. **Gmail integration:**
   - Does the PRD acknowledge Gmail's UI constraints?
   - Are there assumptions about UI real estate, navigation patterns, or interactions that may not work within Gmail?

4. **Technical signals:**
   - Any requirements that seem technically risky? (real-time sync, large data volumes, third-party dependencies)
   - Not your job to solve, but flag for engineering discussion

### Phase 7: Edge Cases & Rollout

1. **Edge cases to check for:**
   - What happens at scale? (100+ agents, 10,000+ tickets/day)
   - What about permissions? (who can see/do what?)
   - What about migration? (existing data, existing workflows)
   - What about errors? (API failures, timeouts, invalid data)
   - What about conflicts? (two agents working the same ticket)

2. **Rollout considerations:**
   - Feature flag / gradual rollout?
   - Migration needed for existing users?
   - Documentation / changelog / in-app discovery?
   - Rollback plan if something goes wrong?

### Phase 8: Produce the Review

Output the review using the CPO agent's Review Output Format. Follow it exactly — the command coordinator expects this structure.

Key principles for the review:
- **Be specific** — cite exact sections of the PRD when flagging issues
- **Propose alternatives** — don't just say what's wrong, say what to write instead
- **Prioritize** — Critical Gaps must be fixed before design. Concerns can be addressed in parallel
- **Include competitive context** — ground your feedback in market reality
- **Assign a verdict** — APPROVED, REFINE, or RETHINK

## What NOT to Do

- Don't rubber-stamp a well-written PRD that solves the wrong problem
- Don't rewrite the PRD — flag gaps and suggest additions
- Don't evaluate design or implementation — that's for later stages
- Don't skip competitive research because the feature "seems straightforward"
- Don't assume a feature is needed just because a competitor has it
- Don't propose solutions — evaluate the problem definition and scope
