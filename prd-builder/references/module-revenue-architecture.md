# Deep-Dive Module: Revenue Architecture
**Parent skill:** `prd-builder/SKILL.md`
**Offered via:** `deep-dive-modules.md`

---

## When to Use

Offered when the product needs a credible path to revenue, especially when the
user has specific financial targets. Skip for hobby projects or internal tools.

## Interview Process

### Step 1: Revenue Intent
- Lifestyle business, venture-scale, or in between?
- First revenue milestone that matters? ($1K MRR? $10K?)
- Timeline pressure for revenue?
- Existing revenue from related work?

### Step 2: Monetization Model Selection

| Model | Best When | Risk |
|-------|-----------|------|
| SaaS Subscription | Ongoing value, recurring usage | Churn kills you |
| Usage-Based | Value scales with consumption | Unpredictable revenue |
| Freemium | Large market, viral potential | Free tier cannibalization |
| One-Time Purchase | Complete solution, low ongoing cost | No recurring revenue |
| Marketplace Fee | Facilitating transactions | Chicken-and-egg problem |
| API/Platform Fee | Developers build on you | Long sales cycle |

### Step 3: Pricing Architecture
Design tier structure. For each tier: name, price, feature gating, upgrade
trigger. Key principles:
- Free tier creates desire to upgrade, not force
- Usage limits > feature removal
- Business tier makes Pro feel reasonable (anchoring)

### Step 4: Unit Economics

**CAC** — Acquisition channels and estimated cost per user.
**LTV** — ARPU / monthly churn rate. Target LTV:CAC ratio ≥ 3:1.
**Conversion** — Free-to-paid rate assumption (B2B SaaS freemium: 2-5%).

**Revenue Projections by Version:**

| Version | Phase | Users | Paying | MRR | Goal |
|---------|-------|-------|--------|-----|------|
| v1.0 | Validation | 50-200 | 5-15 | $X-Y | Validate willingness to pay |
| v2.0 | Growth | 500-2K | 50-150 | $X-Y | Repeatable acquisition |
| v3.0 | Scale | 2K-10K | 200-800 | $X-Y | Predictable unit economics |
| v4.0+ | Expansion | 10K+ | Multi-stream | $X ARR | Compounding revenue |

### Step 5: Revenue Expansion Strategy
- Upsell paths mapped to product usage milestones
- Expansion revenue (seat-based, usage overages, add-ons)
- Adjacent revenue (marketplace, services, data insights)
- Version-mapped revenue unlocks

### Step 6: Financial Milestones
Concrete targets tied to versions: first paying customer, ramen profitability,
break-even, growth investment threshold, scale target.

## Output Locations in PRD
- Section 1.6 (Success Metrics) — revenue and conversion metrics
- Section 2.1 (Feature Set) — feature gating informs priority
- Section 6.1 (Version Roadmap) — revenue milestones per version
- Section 7.2 (Pricing Model) — detailed tier structure
- Section 7.3 (Growth Levers) — expansion revenue strategy
- Section 8.1 (Known Risks) — unit economics assumptions as risks

## Maintenance
Revisit at each major version: conversion rates, churn, pricing validation,
new revenue opportunities, competitor pricing changes.
