# Refine Checklist

Run this checklist against any existing full spot page before starting refinement work.
Check each item — flag what's missing, don't skip items that seem unlikely.

---

## Section Completeness

- [ ] **Safety zone** — `safetyInfo` export in data.ts + Safety section in page.tsx
  - Hazard summary (specific named hazards, not generic warnings)
  - Rescue services (what exists, who operates it)
  - Travel advisory (level + source URL)
  - Medical facilities (nearest hospital/clinic with location)
  - Emergency contacts (coast guard, local emergency number)

- [ ] **Travel-specific zone** — `travelInfo` export in data.ts (and rendered if significant)
  - Visa summary by major passport holders
  - Currency + ATM availability
  - Payment norms (cash vs card at camps)
  - Language (official + practical)
  - Best travel month (single recommendation with reason)
  - Tipping norms

- [ ] **Book Your Trip section** — `id="book"` section exists with camp card grid
  - Camps filtered by `bookingUrl`
  - Browse all links (Booking.com, TripAdvisor, Airbnb — destination-filtered URLs)
  - Fly block (IATA code, routing, kite bag policy, Aviasales link)
  - Protect block (key risk + World Nomads link)

- [ ] **KTP Edge section** — `id="differentiation"` with `ktpDifferentiation` data
  - 6–12 entries
  - Every entry names something specific that competitors omit

---

## Camp Card Quality

- [ ] Every bookable camp has `rating` and `reviewCount` from Google Maps scraper
- [ ] Every bookable camp has `nightlyRate` (or explicitly null — no guesses)
- [ ] Every bookable camp has `googleMapsUrl`
- [ ] Every bookable camp has `photoUrl` pointing to a real file in `public/camps/`
- [ ] Photo files have correct extensions (`.webp` if WebP content, `.jpg` if JPEG)
- [ ] Card wrapper is `<div>` not `<a>` (multiple CTAs = invalid nesting)

---

## Copy Quality Audit

Flag every instance of the following. Do not leave any unflagged.

**Generic adjectives — replace with specific:**
- "amazing", "stunning", "breathtaking", "incredible", "beautiful" → describe what specifically
- "great", "excellent", "fantastic", "perfect" → what makes it great?
- "unique" → in what way?
- "world-class" → by what measure?

**Unnamed specifics — add names:**
- "a local restaurant" → name it
- "nearby camps" → list them
- "some operators" → name them
- "the local cuisine" → name dishes
- "local guides" → name them if known

**Passive constructions that hide information:**
- "kitesurfing is popular here" → who kites here, when, why
- "conditions are good" → what conditions, what time of year, what skill level
- "the water is warm" → temperature in °C/°F, which months

**Competitor copy patterns to eliminate:**
- Any sentence that could apply to 10+ other kite destinations
- Any claim without a named source or specific fact

---

## Data Accuracy

- [ ] All 12 months present in `monthlyWindData` (including off-season with explanation)
- [ ] `kiteSizeGuide` covers peak, shoulder, and off-season (3–5 entries)
- [ ] Coordinates verified via Playwright for all kite spots (`coordinatesPending: false`)
- [ ] HITL gaps list has ≥8 specific questions
- [ ] `verifiedFacts` entries have source URLs, not just claims

---

## Missing AI-Inferred Content

After adding Safety and Travel-specific zones via AI inference, flag all inferred content:

- [ ] Check data.ts for `// AI-INFERRED — verify` comments
- [ ] Prioritize verification of: travel advisory level, rescue services, medical facilities
- [ ] Country-level facts (visa, currency, language) — lower verification priority, generally accurate

---

## Refinement Priority Order

When refining a single page, work in this order:

1. **Add Safety zone** (highest impact — entirely missing from all existing pages)
2. **Add Travel-specific zone** (high impact — entirely missing from all existing pages)
3. **Camp card audit** (scraper run if rating/nightlyRate/Maps link missing)
4. **Copy audit** (flag and fix generic descriptions)
5. **Section completeness** (add Book Your Trip if missing)
6. **Data accuracy** (monthly wind table, coordinates)

Do not start copy audit before structural gaps are filled — structural issues compound copy problems.

---

## After Refinement

- [ ] `npm run build` passes with no type errors
- [ ] All sections render in dev
- [ ] Safety section visible and populated
- [ ] Camp cards: photo + rating + nightlyRate (or priceRange fallback) + Maps link
- [ ] No `// AI-INFERRED` comments left un-verified (or deliberately deferred with a note)
- [ ] Update `app/admin/spots/page.tsx` if spot status changed (partial → complete)
