---
name: ktp-spot-page
description: >
  Build, shell, or refine a Kite the Planet spot page. Use when the user says
  "build a spot page", "add [destination]", "shell page for [X]", "new spot
  page", "refine [spot]", "raise the bar on [spot]", "replicate the Dakhla
  page for [X]", or names a kite destination they want on the platform.
  Presents a mode selector at the start of every session.
---

# KTP Spot Page Builder

## Modes

At the start of every session, ask:

> "Which mode?
> 1. **Full** — complete research-backed spot page
> 2. **Shell** — minimum viable skeleton page + Supabase seed for TIE testing
> 3. **Refine** — raise an existing full page to the current standard"

Do not assume a mode. Wait for explicit selection.

---

## Canonical Standard

Every decision made in this skill should be measured against:
- **`app/spots/dakhla/`** — canonical data.ts + page.tsx (most complete full page)
- **`docs/plans/2026-03-09-spot-page-design.md`** — 33-field spec (the complete vision)

Read both before starting Phase 5 or 6 in Full mode, or Phase 2 in Refine mode.

---

## Mode 1: Full Spot Page

A research-backed page with all data, sections, camp cards, coordinates, and Supabase seed.

### Reference files — read before the phase that needs them

| Reference | Read When |
|-----------|-----------|
| `docs/agents/research/spot-research-dispatch-template.md` | Phase 2 — dispatching research |
| `references/data-ts-guide.md` | Phase 5 — building data.ts |
| `references/camp-card-build.md` | Phase 4 — camp data enrichment |
| `references/page-tsx-structure.md` | Phase 6 — building page.tsx |

### Phases

**Phase 1 — Confirm destination**
Collect before starting:
1. Destination name (what the kite community calls it)
2. Country
3. Spot type — single spot or regional grouping?
4. Sub-spots (if regional) — every named spot to cover
5. Research package exists? — check `docs/research/` first

One question at a time. If a research package exists, skip Phase 2–3.

**Phase 2 — Dispatch research**
Read `docs/agents/research/spot-research-dispatch-template.md` in full.
Fill in DESTINATION, COUNTRY, SPOT TYPE, OUTPUT FILE (`docs/research/<slug>-research-package.md`).
Dispatch 5–6 parallel subagents. Wait for all to complete.

**Phase 3 — HITL gate**
Present a one-paragraph summary: searches, sources, key findings, gaps flagged.
Ask: "Ready to build, or should we fill any gaps first?"
Do not proceed without explicit go-ahead.

**Phase 4 — Camp card data**
Read `references/camp-card-build.md`.
Run scraper → paste output → source photos → verify file extensions.

**Phase 5 — Build data.ts**
Read `references/data-ts-guide.md`.
All 13 exports required. Safety zone and Travel-specific zone are required — not optional.
Flag any AI-inferred content with `// AI-INFERRED — verify` comment.

**Phase 6 — Build page.tsx**
Read `references/page-tsx-structure.md`.
10 sections required. Safety and Travel-specific zones must be wired.

**Phase 7 — Coordinate verification**
Run Playwright lookup for every kite spot (see CLAUDE.md "Coordinate Verification Workflow").
`coordinatesPending: false` for all remotely verifiable spots.

**Phase 8 — Supabase seed**
Seed all fields including Safety + Travel zones.
Schema reference: `docs/plans/2026-03-10-supabase-schema-design.md`.

**Phase 9 — Verify**
- `npm run build` passes with no type errors
- All 10 sections render in dev
- Camp cards show: photo, rating, nightlyRate, Maps link
- Mini maps load on all kite spot cards

---

## Mode 2: Shell Spot Page

A minimum viable page for TIE routing tests. Seeded to Supabase. Clean visual with a shell banner. Sections without data are omitted — no placeholder copy.

### Reference files

| Reference | Read When |
|-----------|-----------|
| `references/shell-page-spec.md` | Before Phase 2 — understand minimum fields and Supabase schema |

### Minimum required fields (6)
1. Coordinates (lat, lng)
2. Country
3. Region
4. Wind season (month range, e.g. "May–October")
5. Skill level range (beginner / intermediate / advanced / all levels)
6. Disciplines (array: freestyle, freeride, wave, foil, wing, lessons)

### Data sources (in priority order)
1. `tools/gmaps-scraper/data/spots-input.csv` — coordinates for many spots
2. `app/admin/spots/page.tsx` — 198-spot master list (slugs, names, regions)
3. AI inference from spot name + country — no research needed at shell tier

### Phases

**Phase 1 — Confirm spot**
Check `app/spots/` — does the directory exist? If yes, check for existing data.ts.
If data.ts exists already, confirm whether to overwrite or extend.

**Phase 2 — Pull minimum fields**
Read `references/shell-page-spec.md`.
Pull from gmaps CSV or AI-infer. Document source for each field.

**Phase 3 — Create data.ts**
Shell exports only (spotMeta + heroStats + kiteSpots skeleton).
No camp data, no full wind tables, no activity/food/logistics detail.

**Phase 4 — Create page.tsx**
Shell layout: hero + mini map + kite spots (summary only) + shell banner.
Banner copy: "Shell page — full research guide coming soon."

**Phase 5 — Supabase seed**
Seed the 6 minimum fields only.

**Phase 6 — Update admin**
In `app/admin/spots/page.tsx`: move spot from Pending → WIP tier.
(Add WIP tier to admin page if it doesn't exist yet.)

### Batch mode
When user says "build shell pages for all pending spots":
1. Read full pending list from `app/admin/spots/page.tsx`
2. Pull coordinates from gmaps CSV for each
3. Process spots in sequence — one data.ts + page.tsx + Supabase seed per spot
4. Report completion status after each

---

## Mode 3: Refine Existing Page

Raise a full page to the current standard. The two confirmed gaps across all 49 existing full pages are the Safety zone and Travel-specific zone.

### Reference files

| Reference | Read When |
|-----------|-----------|
| `references/refine-checklist.md` | Phase 2 — running the checklist |
| `references/page-tsx-structure.md` | Phase 5 — wiring new sections |
| `references/camp-card-build.md` | Phase 7 — camp card audit |

### Phases — single page

**Phase 1** — Identify target page. Read its data.ts and page.tsx.

**Phase 2** — Run checklist from `references/refine-checklist.md`. Report every gap found.

**Phase 3** — Add Safety zone to data.ts.
Source: research package if it exists, otherwise AI inference.
Flag AI-inferred with `// AI-INFERRED — verify` comment.

**Phase 4** — Add Travel-specific zone to data.ts.
Visa, currency/payment norms, language, best travel month.
This is country-level fact — AI inference is accurate without a research package.

**Phase 5** — Update page.tsx with new sections wired. Read `references/page-tsx-structure.md`.

**Phase 6** — Copy audit. Flag and fix: generic adjectives ("amazing", "stunning"), unnamed specifics ("local dishes", "a nearby restaurant"), passive constructions.

**Phase 7** — Camp card audit. If `rating`, `nightlyRate`, `googleMapsUrl` missing — run scraper. Read `references/camp-card-build.md`.

### Batch path (recommended before building shell pages)

Run once to lock the standard across all 49 full pages:
1. Refine Dakhla manually (Phases 1–7) — proves the pattern, locks the structure
2. Batch-apply Safety + Travel zones to all 48 remaining full pages using AI inference
3. Flag all AI-inferred content for spot-check review
4. Shell pages can now be built against a clean, complete standard
