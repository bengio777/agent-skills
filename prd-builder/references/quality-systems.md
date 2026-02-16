# Quality Systems — Deduplication, Cross-Referencing, Validation
**Parent skill:** `prd-builder/SKILL.md`

Read this file during Phase 3 (generation) and Phase 4 (output validation).

---

## 1. Deduplication Logic

### During Discovery (Phase 1)
As the feature brain dump grows, watch for overlapping capabilities described
differently. Common patterns:
- Synonym clusters: "notifications" / "alerts" / "reminders"
- Scope overlap: "client management" / "contact list" / "address book"
- Embedded features: "dashboard" containing "analytics" + "overview" + "stats"

When detected, consolidate:
1. Name the canonical feature (pick the most descriptive term)
2. Confirm with the user: "It sounds like X, Y, and Z are all part of a
   single [canonical name] — agree?"
3. Note the aliases in the glossary so they're recognized if used later

### During Generation (Phase 3)
Before writing the feature set, review the full list and verify:
- No two Feature IDs describe the same capability
- No feature is a subset of another without being called out as such
- Sub-features are nested under parent features, not listed independently

### During Iteration (Phase 5)
When updating a PRD, check that new features don't duplicate existing ones
under different names. Cross-reference the glossary.

---

## 2. Feature ID System

Every feature gets a persistent ID at the moment it enters the Version
Placement Map (Phase 2). Format: **F001, F002, F003...**

Rules:
- IDs are assigned once and never reused, even if a feature is removed
- If a feature is split into sub-features, use F001a, F001b
- If features are merged, retire the secondary ID and note it in the changelog
- IDs persist across all PRD versions — F001 in v1.0 is the same F001 in v3.0
- Use Feature IDs as the primary reference throughout the document: acceptance
  criteria, dependency graphs, build order, roadmap

---

## 3. Cross-Reference Integrity

The PRD has several interconnected reference systems. Before finalizing,
verify all connections are intact:

### Persona → Feature Traceability
- Every Tier 1 persona must be served by at least one P0 feature
- Every P0 feature must reference at least one persona by name
- No feature should reference a persona not in the registry

### Job → Feature Traceability
- Every P0 feature must trace to at least one job from Section 1.3
- Every job must be served by at least one P0 feature
- If a feature doesn't serve a job, question whether it belongs

### Feature → Version Consistency
- Every feature listed in Section 2.1 must appear in the correct version
  column of the Version Roadmap (Section 6.1)
- The Version Roadmap should not reference features that aren't in Section 2

### Glossary Enforcement
- Every domain term defined in Section 1.7 must be used consistently
  throughout the document
- If a term appears in the PRD but not the glossary, add it
- Flag any section where a glossary term is used with a different meaning

### Decision Log Completeness
- Every technology choice in Section 4.2 should have a corresponding
  decision log entry in Section 4.7
- Architecture decisions referenced in Section 6.2 (evolution map) should
  trace back to the decision log

---

## 4. Assumption Risk Ranking

Section 2.3 ranks assumptions on a 2×2 matrix:

```
                    Low Validation Cost    High Validation Cost
High Impact     →   VALIDATE FIRST         PLAN CAREFULLY
Low Impact      →   VALIDATE IF EASY       ACCEPT & MONITOR
```

"VALIDATE FIRST" assumptions are the most important output of this section.
They may reshape v1 scope: instead of building the feature, build the
validation. For example, if the riskiest assumption is "users will pay $15/mo,"
the v1 action is a landing page test, not a payment integration.

---

## 5. Validation Prompts

Before presenting the PRD, surface the 3 riskiest assumptions:

"Before we finalize, here are three assumptions baked into this PRD that could
be wrong. If any of these are off, it changes what we should build first:"

1. [Highest-risk assumption from Section 2.3]
2. [Second highest]
3. [Third highest]

"Do these feel right, or should we stress-test any of them?"

This is not optional — always surface these before delivering the final PRD.

---

## 6. Narrative Coherence Check

After generating the full PRD, do a final pass as if reading it cold:

1. **Problem → Solution flow**: Does the problem statement lead logically to
   the proposed solution? Would a stranger understand why this product exists?

2. **Jobs → Features alignment**: Do the features actually serve the stated
   jobs? Are there jobs with no features, or features with no jobs?

3. **Personas → Scope match**: Are we building for the right personas in v1?
   Does the scope match their pain intensity?

4. **Architecture → Features support**: Can the stated architecture actually
   deliver the stated features? Are there features that require capabilities
   not in the tech stack?

5. **Timeline → Scope realism**: Is the build timeline realistic for the
   feature count and team size? Would a builder look at this and say "no way"?

6. **Roadmap → Vision coherence**: Does the v1 → v2 → v3 progression tell a
   coherent growth story? Does each version logically build on the last?

If any check fails, flag it and revise before presenting.

---

## 7. Module Integration Markers

Several PRD sections can be enriched by optional deep-dive modules. When
generating, check if module output is available and integrate it:

| PRD Section | Enriched By Module | What It Adds |
|---|---|---|
| 1.2 Proposed Solution | Competitive Analysis | Positioning informed by white space |
| 1.4 Value Proposition | Competitive Analysis | Differentiator backed by evidence |
| 2.1 Feature Set | Competitive Analysis | Table stakes inform P0 priority |
| 4.5 Integrations | Platform & Integration | Detailed per-version integration plan |
| 4.4 API Design | Platform & Integration | Platform-trajectory-informed API |
| 6.1 Version Roadmap | All modules | Competitive, platform, and revenue context |
| 7.2 Pricing Model | Revenue Architecture | Detailed tier structure |
| 7.3 Growth Levers | Revenue Architecture | Expansion revenue strategy |
| 1.6 Success Metrics | Revenue Architecture | Revenue and conversion metrics |

If no module was used, generate these sections with the best available context
from discovery. Modules add depth, not existence.

---

## 8. Version Consistency Check

Before finalizing, run this automated cross-check:

1. List all Feature IDs in Section 2.1 with their assigned version
2. List all features referenced in Section 6.1 (Version Roadmap)
3. List all features in Section 9.3 (Dependency Graph)
4. List all features in Section 9.2 (Build Order)

Verify:
- Every Feature ID appears in all four locations
- Version assignments are consistent across all four
- No orphaned features (in one list but not others)
- No phantom features (referenced but never defined)

Flag any inconsistency and resolve before presenting.
