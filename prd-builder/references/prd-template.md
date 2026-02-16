# PRD Template — Structure Reference
**Parent skill:** `prd-builder/SKILL.md`

This defines the section structure for all generated PRDs. For worked examples
of each section, see `prd-template-examples.md`. For quality rules (dedup,
cross-referencing, validation), see `quality-systems.md`.

---

## Document Header

```markdown
# PRD: [Product Name]
**Version:** [X.X]  |  **Date:** [YYYY-MM-DD]  |  **Status:** [Draft | Review | Approved | In Development | Shipped]

## How to Use This PRD
[Implementation handoff — see references/implementation-handoff.md for format]
```

---

## Section 1: Product Overview

### 1.1 Problem Statement
2-3 sentences. Who has the problem, when they experience it, cost of the status
quo. Test: could someone disagree? If not, it's too vague.

### 1.2 Proposed Solution
One paragraph. What the product does, why it's better, the core insight.

### 1.3 Jobs-to-be-Done
2-4 core jobs users are hiring this product to do. Format per job:
- **Job:** [verb phrase — e.g., "Get paid faster for completed work"]
- **Current approach:** How they do it today
- **Pain:** What makes the current approach inadequate
- **Success state:** What "done" looks like

Jobs are the anchor — every feature must trace to a job, or it doesn't belong.

### 1.4 Persona Registry
All personas organized by version tier. For each persona:
- Name, role, version entry point (Tier 1/2/3)
- Narrative user story (3-5 sentences, empathy-driven)
- Pain intensity (nice-to-have vs. hair-on-fire)
- Architectural implications for earlier versions

### 1.5 Value Proposition
One sentence: "For [target user] who [problem], [product] is a [category] that
[key benefit]. Unlike [alternative], it [differentiator]."

### 1.6 Success Metrics & Instrumentation
3-5 metrics. Each needs: what to measure, target at launch + 90 days, and
**how it will be tracked** (specific events, tools, dashboards). Prefer
metrics tied to user value over vanity metrics.

### 1.7 Glossary
Define every domain term used in the PRD. Format:
- **Term**: Definition. Used consistently throughout as [canonical usage].

Establish this early — it prevents "workspace" meaning three different things
in three different sections.

---

## Section 2: Scope & Prioritization

### 2.1 MVP Feature Set
Features organized by priority tier. Each P0/P1 feature gets:
- **Feature ID**: F001, F002, etc. (persists across all versions)
- **Name**: Canonical name (deduplicated)
- **Job(s) served**: Reference to Section 1.3
- **Persona(s)**: Reference to Section 1.4
- **User Story(ies)**: "As a [persona], I want [action], so that [benefit]."
- **Description**: What it does (1-2 sentences)
- **Acceptance Criteria**: Given / When / Then format
- **Failure mode**: What does failure look like for this feature? How detected?
- **Estimated Effort**: T-shirt size (S/M/L/XL) with justification

P2 features: Brief descriptions with Feature IDs only.

### 2.2 Out of Scope
Explicit list of what is NOT in this version. Be specific.

### 2.3 Assumptions — Risk Ranked
Each assumption gets a risk ranking:
- **Assumption**: What must be true
- **If wrong, impact**: How bad (low/medium/high)
- **Validation cost**: How cheaply can this be tested (low/medium/high)
- **Validate before building?**: YES for high-impact + low-validation-cost.
  These reshape v1 scope — build the validation, not the feature.

### 2.4 Kill Criteria
2-3 measurable conditions under which you stop building this product or
pivot significantly. Tied to specific timeframes and metrics.

---

## Section 3: User Experience

### 3.1 User Flows
Critical journeys. For each: trigger, steps, happy path, error states.
Cover at minimum: onboarding, core task, returning user.

### 3.2 Information Architecture
Main screens/pages and hierarchy.

### 3.3 Feedback Loop Design
How you'll learn from users post-launch:
- What's instrumented (events, funnels, session recording)
- Qualitative channels (feedback widget, user interviews, support)
- Cadence: when and how often you review data
- Specific learning goals for v1 (what questions must v1 answer?)

### 3.4 Design Principles (optional)
UX priorities if the product has specific ones.

---

## Section 4: Technical Architecture

### 4.1 System Architecture
Architecture pattern, key components, communication patterns.
For MVPs: prefer simplicity. A monolith that ships beats microservices that don't.

### 4.2 Tech Stack
Concrete technologies in a table with rationale. Justify based on team
capabilities, speed to ship, and scaling needs.

### 4.3 Data Model
Core entities, key fields, relationships, cardinality, constraints.

### 4.4 API Design
Core endpoints: method, path, purpose, key fields, auth requirements.
Focus on P0; P1+ as stubs.

### 4.5 Third-Party Integrations
Per integration: service, purpose, criticality, cost, alternatives.
*Enriched by Platform & Integration module if used.*

### 4.6 Infrastructure & DevOps
Environments, CI/CD, monitoring, logging. Minimal but intentional for MVPs.

### 4.7 Decision Log
Key technical and product decisions with rationale. Format per entry:
- **Decision**: What was decided
- **Alternatives considered**: What else was evaluated
- **Rationale**: Why this choice
- **Revisit if**: Conditions that would trigger reconsideration

---

## Section 5: Cross-Cutting Concerns

### 5.1 Authentication & Authorization
Login methods, roles/permissions, token management.

### 5.2 Security
Encryption, input validation, rate limiting, OWASP considerations, compliance.

### 5.3 Performance & Scaling
Expected traffic, bottlenecks, caching, database scaling approach.

### 5.4 Error Handling & Resilience
Third-party outage handling, user experience during errors, retry strategies.

---

## Section 6: Product Lifecycle Roadmap

### 6.1 Version Roadmap
Table: Version, Theme, Target Personas (by tier), Key Capabilities, Scaling
Stage. *See `prd-template-examples.md` for format.*

### 6.2 Architecture Evolution Map
Per version transition: what stays, what bends, what breaks.

### 6.3 Technical Debt Budget
Per version: intentional debt taken, debt inherited, debt being paid down.

### 6.4 Version Retrospective (v2.0+ only)
What shipped vs. planned, user feedback, technical learnings, strategic shifts.

### 6.5 Scaling Checkpoints
Three-dimensional readiness: technical, product, operational.

---

## Section 7: Go-to-Market (optional but recommended)

### 7.1 Launch Strategy
Launch type, channels, launch checklist.

### 7.2 Pricing Model
Tiers, feature gating, price points. *Enriched by Revenue Architecture module.*

### 7.3 Growth Levers
Organic growth drivers, first paid acquisition channel to test.

---

## Section 8: Risks & Open Questions

### 8.1 Known Risks
Per risk: description, impact, likelihood, mitigation.

### 8.2 Open Questions
Per question: the decision, known options, deadline, owner.

---

## Section 9: Timeline & Milestones

### 9.1 Development Phases
Per phase: name, duration, deliverables, dependencies.

### 9.2 Build Order (within phases)
Logical sequence of what to build each day/week within a phase. Actionable
enough that a builder knows what to work on *today*.
*See `references/implementation-handoff.md` for chunking guidance.*

### 9.3 Dependency Graph
Text-based map showing which features block which. Reveals the critical path.
*See `references/implementation-handoff.md` for format.*

### 9.4 Launch Criteria
Minimum bar for launch. What must be true? What can be imperfect?

---

## Section 10: Changelog & Decision Trail

### Changelog
Track what changed between versions: Added, Changed, Removed, Decision.

### Decision Log Appendix
Running log of decisions across all versions. Newest entries first.
