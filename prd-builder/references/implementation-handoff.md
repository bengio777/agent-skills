# Implementation Handoff — Build Chunking, Dependencies, Handoff
**Parent skill:** `prd-builder/SKILL.md`

Read this file during Phase 4 (Output) to generate the handoff sections.

---

## 1. "How to Use This PRD" Header

Every generated PRD starts with this section immediately after the document
header. Adapt the content based on who the user said would read the PRD:

```markdown
## How to Use This PRD

**For solo builders:** Start with Section 9 (Timeline) for build order —
it tells you what to work on today. Reference Section 4 (Architecture)
for stack decisions. Use Feature IDs (F001, F002...) to trace any feature
back to its job, persona, and acceptance criteria.

**For dev teams:** Section 9.3 (Dependency Graph) shows the critical path.
Assign parallel workstreams based on independent feature branches. Each
feature's acceptance criteria (Section 2.1) are the definition of done.

**For AI coding agents (Claude Code, Cursor, etc.):** Feed the full PRD as
context. Start with the data model (4.3), then implement features in the
build order (9.2). Acceptance criteria are your test cases. The glossary
(1.7) defines domain terminology.

**For stakeholders/investors:** Sections 1 and 6 cover vision and roadmap.
Section 7 covers go-to-market and pricing.
```

---

## 2. Build Order Chunking (Section 9.2)

Break each development phase into day-level or week-level chunks. The goal:
a builder opens the PRD and knows what to do *right now*, not just this month.

### Chunking Principles

**Infrastructure before features.** Data model, auth, and deployment pipeline
come first. You can't build features on nothing.

**Dependency order.** Use the dependency graph (Section 9.3) to sequence
features. Never schedule a feature before its dependencies are complete.

**Vertical slices over horizontal layers.** Build one complete feature
(DB → API → UI) before starting the next. A half-built feature across three
layers is worth nothing; one fully working feature is demoable and testable.

**Hardest uncertainty first.** If a feature involves technical risk (novel
integration, unproven approach), build it early. Don't save the risky stuff
for the end when there's no time to pivot.

**Feedback points.** Schedule natural checkpoints where the builder can show
something to a user or test an assumption. Don't go 4 weeks without
validating that what you're building is wanted.

### Format

```
v1.0 — Phase 1: Foundation (Week 1-2)
  Day 1-2:  [infrastructure task] — [what it enables]
  Day 3-4:  [infrastructure task] — [what it enables]
  ...

v1.0 — Phase 2: Core Features (Week 3-5)
  Day X:    F001 [feature name] — [which parts: DB, API, UI]
  Day X:    F002 [feature name] — [depends on F001 being complete]
  ...
  ★ CHECKPOINT: Demo core flow to [target user/tester]

v1.0 — Phase 3: Polish & Launch (Week 6)
  Day X:    Error handling, edge cases, input validation
  Day X:    Mobile responsiveness, loading states, empty states
  Day X:    Deploy, monitoring, launch checklist
```

---

## 3. Dependency Graph (Section 9.3)

A text-based visualization of which features block which others. This reveals
the **critical path** — the longest chain of dependent features that determines
the minimum possible timeline.

### Format

```
F001 [Feature Name]
  └── F002 [Feature Name] (blocked by F001)
       └── F004 [Feature Name] (blocked by F002)
F003 [Feature Name]
  └── F005 [Feature Name] (blocked by F001 + F003)

CRITICAL PATH: F001 → F002 → F004 (3 features, estimated X days)
PARALLEL TRACK: F003 can start alongside F001
```

### How to Build the Graph

1. For each feature, ask: "What must exist before this can be built?"
2. Draw the chains
3. Identify the longest chain (critical path)
4. Identify features with no upstream dependencies (can start immediately)
5. Identify features that can be built in parallel (independent chains)

### What It Reveals

- **Bottleneck features:** Features that block the most downstream work.
  These get top priority and the best engineering attention.
- **Parallel opportunities:** Independent features that can be built
  simultaneously (relevant for teams, or for solo builders alternating
  when blocked on external dependencies like API approvals).
- **Risk concentration:** If the critical path runs through a technically
  risky feature, that's where timeline risk lives. Address it early.

---

## 4. Feature Failure Modes (Section 2.1)

Each P0 feature should briefly describe what product failure looks like and
how it would be detected. This is not technical error handling (Section 5.4)
— it's product-level failure.

### Format per Feature

> **Failure mode:** [What happens if this feature doesn't achieve its purpose]
> **Detection:** [How you'd notice — metric, behavior pattern, user feedback]
> **Response:** [What you'd do — investigate, redesign, cut, pivot]

### Common Failure Patterns

- Users engage with the feature but don't complete the intended action
- Users ignore the feature entirely (zero adoption)
- Users use the feature but in an unexpected way that doesn't serve the job
- Feature works technically but doesn't reduce the pain it was designed for
- Feature creates new problems (complexity, confusion, support burden)

---

## 5. Feedback Loop Design (Section 3.3)

This is the learning infrastructure built into v1. It answers: "How will we
know if this is working, and how will we know what to build next?"

### Quantitative Instrumentation

List specific events to track. Tie each to a feature and a learning goal:

| Event | Feature | Learning Goal |
|-------|---------|--------------|
| `invoice_created` | F001 | Are users creating invoices? |
| `invoice_sent` | F001+F002 | Are they actually sending them? |
| `invoice_sent / invoice_created ratio` | F001+F002 | Is there send friction? |
| `payment_received` | F003 | Is the payment flow working? |
| `time_from_create_to_send` | F001+F002 | How fast is the core workflow? |

### Qualitative Channels

- In-app feedback widget (low friction, captures context)
- Scheduled user interviews: 5 users in weeks 2-3 post-launch
- Support channel monitoring (email, chat, social)
- Session recording for onboarding funnel analysis

### Learning Goals for v1

Frame as specific questions v1 must answer:
1. "Do users actually send the invoices they create?" (validates core loop)
2. "What's the #1 thing users ask for that we didn't build?" (informs v2)
3. "Where do users drop off in onboarding?" (retention risk)
