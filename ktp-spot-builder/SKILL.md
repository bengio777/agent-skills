---
name: ktp-spot-builder
description: >
  KTP spot data rulebook. Fires automatically when editing any file in app/spots/,
  including data.ts, page.tsx, or any spot-related TypeScript file. Encodes all
  mandatory display rules, structural constraints, and quality standards for KTP
  spot pages — coordinates, temperature format, mini map placement, copy quality,
  and canonical structure. Use this skill whenever touching spot data or spot page
  files, even for small edits. The rules here are non-negotiable and exist because
  past violations caused inconsistencies across 49+ live pages.
metadata:
  filePattern: "app/spots/**,**/spots/**/*.ts,**/spots/**/*.tsx,**/data.ts,**/spots/*/page.tsx"
  priority: 90
---

# KTP Spot Builder — Standing Rulebook

> Canonical reference: `app/spots/dakhla/data.ts` + `app/spots/dakhla/page.tsx`
> Read these before making structural changes to any spot page.

---

## 1. Coordinates — mandatory Playwright lookup

**Never guess or infer coordinates.** Every kite spot requires a verified lat/lng via Playwright.

```js
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage();
  await page.goto('https://www.google.com/maps/search/' + encodeURIComponent('SPOT NAME'));
  await page.waitForTimeout(5000);
  try { await page.click('a[href*="/maps/place/"]', { timeout: 2000 }); await page.waitForTimeout(3000); } catch(e) {}
  const url = page.url();
  const match = url.match(/@(-?\d+\.\d+),(-?\d+\.\d+)/);
  console.log(match ? match[1] + ',' + match[2] : 'NOT FOUND');
  await browser.close();
})();
```

- Search by specific spot name (e.g. "Kite Beach Dakhla"), not just the destination
- `coordinatesPending: false` for all remotely verifiable spots
- `coordinatesPending: true` only for genuinely unverifiable spots (unnamed offshore reefs, etc.)
- If a user provides a Google Maps URL, extract `@lat,lng` from it

---

## 2. Temperature — always dual-unit

Every temperature value anywhere on the site must show both units.

| Format | Example |
|--------|---------|
| Single value | `22°C / 72°F` |
| Range | `18–22°C / 64–72°F` |

This applies to: water temp labels, wind data notes, gear recommendations, trip builder steps — everywhere. Never show a temperature in only one unit, even in data.ts comments.

---

## 3. Mini map — placement and pattern

**Placement rule (non-negotiable):** The mini map must appear immediately after the title/skill-badge header block and before the description in every kite spot card.

Card order: `title + badge` → `mini map` → `shortDescription` → disciplines → hazards → access

**Never** place the mini map at the bottom of a card.

**Pattern:**
```tsx
{spot.coordinates && (
  <div className="relative rounded-lg overflow-hidden h-36 -mx-1">
    <iframe
      src={`https://www.google.com/maps/embed/v1/place?key=${process.env.GOOGLE_MAPS_EMBED_KEY}&q=${spot.coordinates}&zoom=13`}
      width="100%" height="100%"
      style={{ border: 0, pointerEvents: spot.coordinatesPending ? "none" : "auto" }}
      loading="lazy" referrerPolicy="no-referrer-when-downgrade"
    />
    {spot.coordinatesPending && (
      <div className="absolute inset-0 flex items-center justify-center" style={{ background: "rgba(9,9,11,0.6)", backdropFilter: "blur(2px)" }}>
        <p className="text-xs text-zinc-400 italic">Coordinates pending: local verification required</p>
      </div>
    )}
  </div>
)}
```

---

## 4. ktpDifferentiation — 3 entries, specific not generic

Every spot's `data.ts` must include at least 3 `ktpDifferentiation` entries. Each must be a specific, verifiable insight that users cannot get from a generic travel site.

**Good:** "Baleal mid-tide timing — sessions only viable 2hrs either side of mid-tide due to sandbar exposure"
**Bad:** "Great kite conditions year-round with warm water and consistent winds"

Flag any entry that uses: "great", "amazing", "stunning", "beautiful", "perfect", "ideal", "world-class" (unless it's a factual designation like WSL World Surfing Reserve).

---

## 5. Structural constants

| Property | Value |
|----------|-------|
| Background color | `#13192a` |
| Mini map height | `h-36` |
| Mini map margin | `-mx-1` |
| Map zoom | `13` |
| `coordinatesPending` default | `false` |

---

## 6. Copy quality flags

Before committing any spot copy, check for:
- **Generic adjectives:** "amazing", "stunning", "beautiful", "perfect", "ideal" — replace with specific facts
- **Unnamed specifics:** "local dishes", "a nearby restaurant", "the local school" — name them
- **Passive constructions:** "it is known for", "is considered" — rewrite as direct statements
- **Unverified superlatives:** "one of the best", "most popular" — only use if sourced

---

## 7. Page.tsx structure — 10 required sections

In order:
1. Hero (full-width image + title + skill badges)
2. Mini map (immediately after hero — see rule 3)
3. Overview / short description
4. Wind & conditions
5. Kite spots (individual spot cards, each with their own mini map)
6. Camp cards (accommodations)
7. Safety zone
8. Travel-specific zone (visa, currency, language, best travel month)
9. Operators directory (if applicable)
10. Book Your Trip CTA

Sections 7 and 8 are required — not optional. If data isn't available, use AI-inferred content and flag with `// AI-INFERRED — verify`.

---

## 8. AI-inferred content

Any field inferred by AI without a source must be flagged:
```ts
waterTempC: 22, // AI-INFERRED — verify
```
This applies to data.ts only — page.tsx structural code doesn't need this flag.

---

## Relationship to ktp-spot-page

This skill is the rulebook. `ktp-spot-page` is the workflow driver.

- Working through a full build/shell/refine session → `ktp-spot-page`
- Editing an existing spot file and need the rules in context → this skill fires automatically
- Both can be active simultaneously — they don't conflict
