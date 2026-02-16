# Deep-Dive Module: Competitive Analysis
**Parent skill:** `prd-builder/SKILL.md`
**Offered via:** `deep-dive-modules.md`

---

## When to Use

Offered during Phase 2 when the user wants strategic clarity on positioning.
Skip for quick-iteration PRDs; use when differentiation matters.

## Interview Process

### Step 1: Define Key Features for the Space
Ask the user to define 5-10 capabilities that any product in this space is
judged on. These are the buyer's comparison criteria, not the user's features.

Prompt: "If someone was evaluating products in this space, what capabilities
would they compare?"

Features should be specific enough to evaluate, meaningful to target personas,
and comparable across products.

### Step 2: Identify Competitors
User names known competitors. Use web search to fill gaps:
- Direct competitors (same problem, same persona)
- Adjacent competitors (different angle, same problem)
- Substitutes (spreadsheets, manual processes, hiring someone)
- Emerging players

Target 4-8 competitors.

### Step 3: Feature-by-Feature Matrix

| Rating | Meaning |
|--------|---------|
| ✅ Strong | Best-in-class or near it |
| 🟡 Adequate | Has it, nothing special |
| ⚠️ Weak | Has it, poorly implemented |
| ❌ Missing | Doesn't offer this |
| 🔒 Paid Only | Behind a paywall |

Build the matrix: features as rows, competitors as columns.

### Step 4: Opportunity Identification
From the matrix, identify:
- **White space** — All competitors weak/missing. Potential differentiators.
- **Table stakes** — Most competitors strong. Must match to compete.
- **Crowded strengths** — Multiple competitors strong. Expensive to compete.
- **Pricing gaps** — Features locked behind premium tiers elsewhere.

### Step 5: Positioning Statement
"In a market where [competitors] focus on [strengths], [product] wins by
[angle] — excelling at [white space] while matching on [table stakes]."

### Step 6: Version-Mapped Competitive Strategy
- **v1**: Match table stakes + deliver 1-2 white space differentiators
- **v2**: Close gaps on crowded strengths where weakest
- **v3+**: Extend lead on differentiators, enter new competitive dimensions

## Output Locations in PRD
- Section 1.2 (Proposed Solution) — positioning
- Section 1.5 (Value Proposition) — evidence-backed differentiator
- Section 2.1 (Feature Set) — table stakes inform P0
- Section 6.1 (Version Roadmap) — competitive strategy per version

## Maintenance
Revisit the competitive matrix at each major version. Competitors evolve —
a v1 white space may become table stakes by v3.
