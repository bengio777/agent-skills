# Iteration & Scaling — Version Progression, Retrospectives, Checkpoints
**Parent skill:** `prd-builder/SKILL.md`

Read this file during Phase 5 (Iteration) and Phase 6 (Scaling Checkpoints).

---

## 1. Minor Iterations (v1.0 → v1.1)

Refinements within a release: reprioritized features, updated acceptance
criteria, architectural pivots, new learnings.

**Process:**
1. Read the current PRD version
2. Identify what's changing and why
3. Verify changes don't break cross-reference integrity (run checks from
   `quality-systems.md` Section 3)
4. Increment the minor version number
5. Add changelog entries with rationale
6. Update the decision log if any decisions changed

---

## 2. Major Version Progressions (v1.x → v2.0)

New production releases driven by user feedback, market shifts, or scaling
needs. Each major version gets its own PRD file.

**Process:**
1. Read the current PRD and any prior retrospectives
2. Run a **Version Retrospective** (Section 3 below)
3. Re-evaluate the Persona Registry — any new personas? Tier changes?
4. Update the Product Lifecycle Roadmap (Section 6 of template)
5. Run applicable deep-dive modules if context has changed
6. Re-assess kill criteria — are we still building the right thing?
7. Draft new version scope informed by retrospective data
8. Re-evaluate technical architecture for scaling needs
9. Run quality systems checks (`quality-systems.md`)
10. Generate new PRD: `PRD-ProductName-v2.0.md`

Previous PRD versions are preserved as-is — never overwritten. Each major
version references its predecessor. This creates a decision trail.

---

## 3. Version Retrospective (required for v2.0+)

Before scoping any new major version, capture what happened. This section
becomes Section 6.4 in the new PRD.

### What Shipped vs. Planned
- Features that shipped as designed
- Features that changed significantly during development (and why)
- Features that were cut (and what triggered the cut)
- Unplanned features that were added (and why)

### User Feedback & Behavioral Data
- Top 3 things users loved (with evidence)
- Top 3 pain points or feature requests
- Surprising usage patterns (things users did that weren't anticipated)
- Key metrics vs. targets from prior version's success metrics
- Feedback loop learnings: what questions did v1 answer?

### Technical Learnings
- What scaled well
- What broke or struggled under load
- Third-party services that underperformed or exceeded expectations
- Performance bottlenecks discovered in production
- Technical debt that's now blocking progress

### Assumption Validation
- Go back to Section 2.3 of the prior PRD
- For each "validate first" assumption: what did we learn?
- Were any assumptions wrong? How did that affect the product?

### Kill Criteria Check
- Review the kill criteria from the prior PRD
- Did any trigger? If close, discuss honestly.
- Update kill criteria for the new version

### Strategic Shifts
- Market changes that affect product direction
- Competitive moves that require response
- New opportunities discovered through prior version learnings

---

## 4. Scaling Checkpoints

Evaluated at each major version boundary across three dimensions.

### Technical Scaling Readiness
- Can the current architecture handle 10x the current load?
- What's the most likely failure point at 10x?
- What's the estimated infrastructure cost at 10x? Are unit economics viable?
- Review the Architecture Evolution Map (Section 6.2) — are we on track?
- Is technical debt blocking scaling? What must be paid down this version?

### Product Scaling Readiness
- Are we deepening value for existing users (retention) or expanding to new
  segments (acquisition)? What's the strategy and why?
- Is the core loop validated enough to build on, or does it need rethinking?
- Feature-to-complexity ratio: are we adding value or bloat?
- Are new personas (Tier 2/3) ready to be served, or is Tier 1 still
  underserved?

### Operational Scaling Readiness
- Can the current team/process support this version's scope?
- What roles become necessary? (hire, outsource, co-founder?)
- Monitoring, alerting, and support infrastructure — keeping up with growth?
- Documentation, onboarding, self-serve support — are these adequate?
- What operational processes need to exist that don't yet?

---

## 5. Architecture Evolution Map Updates

At each major version, update Section 6.2 with:

**What stays** — Components that scale cleanly to the next stage.

**What bends** — Components needing optimization but not replacement
(add caching, read replicas, CDN, indexes).

**What breaks** — Components that fundamentally can't handle the next stage
and need rearchitecture (monolith → service extraction, new data model
patterns, infrastructure migration).

For each "breaks" item, create a decision log entry with the migration plan,
estimated effort, and risk.

---

## 6. Technical Debt Budget Updates

At each major version, update Section 6.3:

- **New intentional debt** — Shortcuts being taken knowingly in this version,
  with a note on when they'll need addressing
- **Inherited debt** — Carried forward from prior versions, with justification
  for why it's still acceptable
- **Debt being paid** — What's being fixed from prior shortcuts, with the
  trigger that made it necessary now

If inherited debt exceeds 30% of the version's engineering effort, flag it
as a scaling risk. Debt compounds; ignoring it doesn't make it smaller.
