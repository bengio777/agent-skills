---
name: prd-builder
description: >
  Generate production-ready Product Requirements Documents (PRDs) for planning,
  building, iterating, and scaling applications. Use this skill whenever the user
  wants to plan an app, create a PRD, write product specs, scope an MVP, define
  product requirements, plan a feature, architect a new product, or says things
  like "I have an app idea", "help me plan this product", "write a PRD",
  "scope this project", "what should I build first", or "help me ship this".
  Also triggers on requests to update, iterate, or version an existing PRD.
  This skill produces structured, actionable documents — not vague wishlists.
---

# PRD Builder — Production-Ready Product Requirements

## Purpose

This skill produces PRDs that bridge the gap between "I have an idea" and "this
is ready to build." Every PRD is structured so a solo founder, a dev team, or an
AI coding agent can pick it up and start shipping immediately.

A PRD is a living blueprint, not a dusty spec. It should be opinionated about
what to build first, honest about what's out of scope, and clear enough that
someone with zero context can understand the product.

## Reference Files

Read the relevant reference file BEFORE executing each phase:

| File | Read When |
|------|-----------|
| `references/prd-template.md` | Phase 3 — generating the PRD |
| `references/prd-template-examples.md` | Phase 3 — for worked examples of each section |
| `references/quality-systems.md` | Phase 3-4 — deduplication, cross-referencing, validation |
| `references/version-coaching.md` | Phase 2 — coaching criteria, placement map format |
| `references/deep-dive-modules.md` | Phase 2 — module index and offer script |
| `references/module-competitive-analysis.md` | When user opts into competitive deep dive |
| `references/module-platform-integration.md` | When user opts into platform/integration deep dive |
| `references/module-revenue-architecture.md` | When user opts into revenue deep dive |
| `references/implementation-handoff.md` | Phase 4 — build chunking, dependency graph, handoff |
| `references/iteration-scaling.md` | Phase 5-6 — version progression, retrospectives, scaling |

## Workflow

### Phase 1: Discovery Interview

Capture the user's full vision — not just what to build first, but where the
product could go. Adapt depth to the user: quick v0.1 or comprehensive deep
dive, either is valid.

**Product Vision** — What problem? Who has it worst? What's the 2-3 year dream?

**Jobs-to-be-Done** — What 2-4 core jobs is the user hiring this product to do?
Jobs are durable ("help me get paid faster"); features are implementation.

**Personas (capture ALL)** — Day-one user in detail. Then: who else eventually?
Power users, admins, API consumers, partners. For each: when do they become
relevant? Do they require v1 architectural decisions?

**Feature Brain Dump** — Everything they've imagined. No filter. For each item:
who is this for, what job does it serve? Actively deduplicate as the list grows —
consolidate overlapping capabilities and confirm canonical names with the user.

**Constraints** — Solo or team? Budget? Timeline? Tech preferences? These
constraints actively limit scope during version coaching.

**Competitive Landscape** — What exists? What's the gap? Use web search when
helpful.

**Monetization** — How and when does this make money?

**Kill Criteria** — Under what conditions would you stop building this? Define
2-3 measurable failure signals (e.g., "can't get 50 users in 60 days").

**Glossary Seeding** — As domain terms emerge, capture them. "Workspace" vs.
"organization" vs. "team" — define these now so the PRD is internally consistent.

### Phase 2: Version Coaching

Read `references/version-coaching.md` for detailed coaching criteria and the
Version Placement Map format.

Read `references/deep-dive-modules.md` for available modules. Offer them
naturally: "I have enough to draft. Want to go deeper on competitive analysis,
integrations, or revenue — or should I draft first and layer those in later?"
Modules are additive, never blocking.

Coach each feature into a version using six criteria: persona dependency,
dependency chain, complexity vs. value, architectural implications, revenue
unlock, and learning value. Present the Version Placement Map and discuss with
the user. Disagreements are where the best decisions happen.

Assign each feature a **Feature ID** (F001, F002, etc.) during coaching. These
IDs persist across the entire PRD and all future versions for reliable
cross-referencing.

### Phase 3: Generate the PRD

Read `references/prd-template.md` for the exact structure. Read
`references/prd-template-examples.md` for worked examples of each section.
Read `references/quality-systems.md` for deduplication, cross-reference
integrity, and consistency rules.

Generate the PRD following the template precisely. Be specific and opinionated.
Every feature needs user stories and acceptance criteria. The architecture must
be concrete enough to start building from.

### Phase 4: Output & Validation

Generate the PRD as a **Markdown file** saved to the current working directory.
Naming: `PRD-[ProductName]-v[X.X].md` (v0.1 for drafts, v1.0 for complete).

Before presenting, run the quality checklist and the narrative coherence check
(see `references/quality-systems.md`). Surface the 3 riskiest assumptions baked
into the document and ask the user to gut-check them.

Read `references/implementation-handoff.md` and include a "How to Use This PRD"
header at the top of the generated document.

### Phase 5: Iteration & Version Progression

Read `references/iteration-scaling.md` for detailed procedures on minor
iterations, major version progressions, retrospectives, and scaling checkpoints.

### Phase 6: Scaling Checkpoints

Evaluated at each major version boundary across three dimensions: technical,
product, and operational scaling readiness. See `references/iteration-scaling.md`.

## Quality Checklist

Before presenting any PRD, verify:

- [ ] Problem statement is specific and testable
- [ ] Core jobs-to-be-done are identified and durable
- [ ] Persona registry covers all tiers with version entry points
- [ ] Features are deduplicated with canonical names and Feature IDs
- [ ] Each P0/P1 feature has user stories and acceptance criteria
- [ ] Glossary defines all domain terms used in the document
- [ ] Feature IDs, persona names, and job references are cross-referenced consistently
- [ ] Technical architecture specifies concrete stack with rationale
- [ ] Architecture accounts for scaling to the next growth stage
- [ ] Data model covers core entities and relationships
- [ ] Dependency graph shows critical path and blocking relationships
- [ ] Kill criteria define measurable failure signals
- [ ] Assumptions are ranked by risk and cheapest-to-validate are flagged
- [ ] Feedback loop design specifies how you'll learn from users post-launch
- [ ] Success metrics are measurable with instrumentation plan
- [ ] Product lifecycle roadmap extends beyond the current version
- [ ] Timeline includes build-order chunking within each phase
- [ ] Implementation handoff section is present and actionable
- [ ] Version roadmap matches features listed in scope (consistency check)
- [ ] Decision log captures key choices with rationale and revisit conditions
- [ ] For v2+: retrospective from prior version informs scope
- [ ] Narrative coherence: document tells a logical story end-to-end

## Tone and Style

Write like a sharp technical PM who respects the reader's time. Be direct.
Use plain language. Avoid corporate filler. When a section requires a judgment
call, make one and explain why.

If the user is a solo builder, optimize for speed and pragmatism. If they're
building for a team, include more context and rationale.
