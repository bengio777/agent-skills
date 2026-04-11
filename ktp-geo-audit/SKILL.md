---
name: ktp-geo-audit
description: >
  KTP Generative Engine Optimization (GEO) audit skill. Runs when the user invokes
  /ktp-geo-audit or asks to check, audit, or improve KTP's LLM search position.
  Reads the current state of KTP spot pages, schema markup, content structure, and
  site configuration, then produces a prioritized improvement report based on the
  GEO strategy in docs/strategy/geo-strategy.md. Tracks progress against prior
  audits. Use after any significant product release or content update.
metadata:
  trigger: manual
  priority: 80
---

# KTP GEO Audit Skill

## Purpose

Audit KTP's current GEO (Generative Engine Optimization) readiness and produce a
prioritized report of improvements. The goal: ensure KTP is the cited source for
kite travel queries across ChatGPT, Perplexity, Claude, and Google AI Overviews.

Reference documents:
- `docs/strategy/geo-strategy.md` — full GEO playbook and standards
- `docs/strategy/llm-search-supremacy.md` — strategic framing
- `docs/strategy/geo-citation-log.md` — prior audit results (create if missing)

---

## Audit Sequence

Run these checks in order. Report findings per section before moving to the next.

---

### Check 1 — LLM Crawler Allowlist

Check `public/robots.txt` or `next.config.ts` for any `Disallow` rules. Verify the
following crawlers are explicitly allowed (or not blocked):

| Crawler | LLM | Must be allowed |
|---|---|---|
| GPTBot | ChatGPT | Yes |
| ClaudeBot | Claude | Yes |
| PerplexityBot | Perplexity | Yes |
| GoogleOther | Google AIO | Yes |
| OAI-SearchBot | ChatGPT search | Yes |

**Pass:** None of these are blocked.  
**Fail:** Any blocked = Priority 1. A silently blocked crawler means KTP never gets
cited by that LLM regardless of content quality. Fix before anything else.

---

### Check 2 — `llms.txt`

Look for `/public/llms.txt` or `llms.txt` in the project root.

**Pass:** File exists and declares KTP's entity, coverage, and key pages.  
**Fail:** File missing — Quick Win, flag as Priority 1. Template is in `docs/strategy/geo-strategy.md`.

---

### Check 3 — Schema Markup Coverage

Always sample these specific spot pages (canonical reference set):
- `app/spots/dakhla/` — canonical reference per CLAUDE.md
- `app/spots/tarifa/` — highest Spain traffic
- `app/spots/cape-verde/` or equivalent — global destination example
- One additional spot chosen at random from `app/spots/`

Check each for:
- `@type: TouristDestination` or `SportsActivityLocation`
- `geo` with coordinates
- `amenityFeature` array (flat water, wave, certifications)
- `sport: Kitesurfing`

Read 1–2 operator listing components. Check for:
- `@type: LocalBusiness`
- `hasCredential` with IKO/VDWS data
- `aggregateRating`

**Score:** X of Y spot pages have schema / X of Y operator pages have schema.  
**Fail threshold:** Any spot page missing schema = flag as Priority 1.

---

### Check 4 — Content Structure, Depth, and Quality

Use the same canonical spot pages sampled in Check 3.

**4a — Table vs. Prose**

| Element | Expected Format | Pass Criteria |
|---|---|---|
| Wind data | Table (month × knots × direction) | Table present |
| Season guide | Table (month × suitability × notes) | Table present |
| Skill level suitability | Structured tags or table | Not buried in prose |
| Water conditions | Structured (flat/wave/chop + depth) | Not vague prose |
| Getting there | Structured steps or table | Findable |

**Score:** X of 5 spot elements structured. Wind in prose = Priority 1. Others = Priority 2.

**4b — Content Depth** (ConvertMate 2026: 4.3x citation multiplier at 20K+ chars; 3.2x at <30 days old)

| Threshold | Target | Pass Criteria |
|---|---|---|
| ≥ 2,500 words per spot page | Required | Estimate from paragraph count |
| ≥ 120 words per H2 section | Required | Spot-check 3 sections |
| ≥ 1 data point per 200 words | Required | Look for stats, numbers, specifics |

**Fail threshold:** Any page under 1,500 words = Priority 1. Under 2,500 = Priority 2.

**4c — Expert Quotes** (+41% visibility — highest single-tactic effect, Aggarwal et al.)

Check each page for at least one direct quote:
- Named person with full name + credential/role
- Specific, verifiable claim (not generic praise)
- Placed in the top half of the page

**Fail:** No named expert quotes = Priority 2.

**4d — Heading Structure**

H2/H3 headings should be phrased as questions where applicable. 68.7% of ChatGPT citations follow logical heading hierarchies (ConvertMate 2026).

**Fail:** All headings are topic labels with no question framing = Priority 3.

---

### Check 5 — FAQ Sections

Read the same spot pages sampled in Check 3. Check for presence of FAQ section with at minimum:
- Best time of year to visit
- Suitable for beginners?
- Wind conditions summary
- IKO/VDWS schools available?

**Pass:** FAQ section exists and covers at least 3 of the 4 questions.  
**Fail:** No FAQ section = Priority 2. Partial = Priority 3.

---

### Check 6 — Query Coverage Map

Check which of these high-value LLM query types KTP currently has content to answer.
For each, identify whether a dedicated page/section exists:

| Query Type | Content Exists? | Page/URL |
|---|---|---|
| "Best kite spots in [region]" | ? | ? |
| "Best kite spots for beginners" | ? | ? |
| "Best kite spots in [month]" | ? | ? |
| "Kite spots with flat water" | ? | ? |
| "IKO certified kite schools in [destination]" | ? | ? |
| "[Destination A] vs [Destination B] for kitesurfing" | ? | ? |
| "Best time to kite in [destination]" | ? | Spot page seasonal section |
| "Kite camps [destination/global]" | ? | ? |

**Score:** X of 8 query types have dedicated content.  
**Fail threshold:** Fewer than 4 covered = Priority 1 content gap.

---

### Check 7 — Spot Page Freshness Signals

Read 3 spot pages. Check for:
- "Last updated" or "Verified [Year]" timestamp on operator listings
- Season data with year reference
- Any stale content signals (school no longer operating, outdated pricing)

**Pass:** Freshness dates present on operator data.  
**Fail:** No freshness signals = Priority 2.

---

### Check 8 — Citation Log Review

Read `docs/strategy/geo-citation-log.md` if it exists. Check:
- When was the last citation audit run?
- Which queries is KTP currently being cited for?
- Which queries is KTP NOT being cited for that it should be?

If the log doesn't exist, flag that the first citation audit needs to be run manually
(instructions: query each target question in ChatGPT search, Perplexity, Claude, and
Google AIO — log whether KTP is cited, what is cited instead).

---

## Report Format

After completing all checks, output the following report:

```
## KTP GEO Audit Report
**Date:** [today]
**Pages sampled:** [list]

### Summary Score
| Check | Status | Priority |
|---|---|---|
| LLM crawler allowlist | PASS/FAIL | P1/P2/P3/— |
| llms.txt | PASS/FAIL | P1/P2/P3/— |
| Schema markup | PASS/FAIL (X/Y pages) | P1/P2/P3/— |
| Content structure — tables | PASS/FAIL (X/5 elements) | P1/P2/P3/— |
| Content depth — word count | PASS/FAIL (avg words) | P1/P2/P3/— |
| Expert quotes | PASS/FAIL | P1/P2/P3/— |
| Heading structure (Q-style) | PASS/FAIL | P1/P2/P3/— |
| FAQ sections | PASS/FAIL | P1/P2/P3/— |
| Query coverage | PASS/FAIL (X/8 types) | P1/P2/P3/— |
| Freshness signals | PASS/FAIL | P1/P2/P3/— |
| Citation log | CURRENT/STALE/MISSING | — |

### Priority 1 — Fix Before Next Deploy
[Specific items with file paths and exact changes needed]

### Priority 2 — Fix This Sprint
[Specific items with file paths and what's missing]

### Priority 3 — Backlog
[Items that would improve GEO but aren't blocking]

### Quick Wins (< 30 min each)
[Items that can be done right now in this session]

### Citation Audit Status
[Last run date, key gaps, next scheduled audit]
```

---

## After the Report

Ask: "Want me to fix the Priority 1 items now?"

If yes — execute them in order: llms.txt first, schema markup second, content structure
third. Commit each as a discrete unit with a GEO-tagged commit message.

Append audit summary to `docs/strategy/geo-citation-log.md` under the current date.
Create the file if it doesn't exist.

---

## Agent Handoff Note (future)

When the GEO agent is built, this skill becomes its audit logic. The agent will:
1. Run this audit automatically after each production deploy
2. Query target questions across AI search engines
3. Compare results to prior audit
4. Post a diff report: what improved, what regressed, what's new to fix
5. File improvement items to the KTP Task Tracker in Notion

Until the agent exists, this skill is the manual equivalent. Run it after every
significant feature release or content update.
