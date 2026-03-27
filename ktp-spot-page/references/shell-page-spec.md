# Shell Page Specification

## Purpose

Shell pages are minimum viable spot pages for TIE (Travel Intelligence Engine) routing tests.
They seed Supabase with 6 core fields and render a clean Next.js page with a shell banner.
They are stepping stones — every shell page will eventually be upgraded to a full page.

---

## Minimum Required Fields (6)

| Field | Type | Example | Source |
|-------|------|---------|--------|
| `coordinates` | `"lat,lng"` string | `"28.729,-17.984"` | gmaps CSV → Playwright lookup |
| `country` | string | `"Spain"` | admin/spots list |
| `region` | string | `"Canary Islands"` | admin/spots list or AI |
| `windSeason` | string | `"May–October"` | AI inference from spot name |
| `skillLevelRange` | string | `"Intermediate–Advanced"` | AI inference |
| `disciplines` | string[] | `["Freeride", "Wave", "Foil"]` | AI inference |

---

## Data Sources (in priority order)

1. **`tools/gmaps-scraper/data/spots-input.csv`** — check for this spot first. Likely has coordinates and basic metadata for many spots.

2. **`app/admin/spots/page.tsx`** — hardcoded array with 198 spots. Has: slug, name, country, region, totalSubSpots, pendingSubSpots, status. Read the array to get region.

3. **AI inference** — for any field not found in the above sources, infer from spot name + country. No research needed at shell tier. Wind season and skill level are well-known for established kite destinations.

**Coordinate rule:** Always run Playwright lookup even if CSV has coordinates — verify the @lat,lng from the actual Maps URL.

---

## data.ts Structure for Shell Pages

Shell pages export a minimal subset — enough for TIE routing, not full editorial.

```ts
// Shell page — data will be expanded when upgraded to full page
// Created: YYYY-MM-DD

export const spotMeta = {
  name: string,
  country: string,
  region: string,
  flag: string,          // emoji — AI can infer from country
  tagline: "",           // empty for shell — filled during full page build
  coordinates: string,   // "lat,lng"
  coastType: "",
  ikoPresence: "",
  windDays: "",
};

export const heroStats = [
  { label: "Wind Season", value: string },     // e.g. "May–Oct"
  { label: "Skill Level", value: string },     // e.g. "Intermediate+"
  { label: "Disciplines", value: string },     // e.g. "Freeride, Wave"
  { label: "Status", value: "Shell Page" },    // always "Shell Page" for shells
];

export type SkillLevel = "All Levels" | "Beginner" | "Intermediate" | "Advanced" | "Intermediate+" | "Intermediate–Advanced";

export interface KiteSpot {
  name: string;
  shortDescription: string;
  skillLevel: SkillLevel;
  disciplines: string[];
  hazards: string;
  tideDependent: boolean;
  access: string;
  coordinates?: string;
  coordinatesPending?: boolean;
}

// At least 1 kite spot entry with basic info (AI-inferred)
export const kiteSpots: KiteSpot[] = [
  {
    name: string,              // primary spot name
    shortDescription: "",      // left empty for shell — filled during full build
    skillLevel: SkillLevel,
    disciplines: string[],
    hazards: "",
    tideDependent: false,
    access: "",
    coordinates: string,
    coordinatesPending: false,
  },
];

// No Camp interface for shell pages — camps section is omitted
// No monthlyWindData, activities, food, logistics, etc.
```

---

## page.tsx Structure for Shell Pages

Shell pages render only the sections where data exists:

```
1. Shell Banner       ← "Shell page — full research guide coming soon"
2. Hero               ← spotMeta + heroStats (wind season, skill level, disciplines)
3. Mini Map           ← coordinates from spotMeta
4. Kite Spots         ← 1+ entries with basic info
```

All other sections (Conditions, Camps, Culture, Food, Logistics, Safety, Book, KTP Edge) are omitted until the page is upgraded to full.

### Shell banner (place immediately below page title, above hero section)

```tsx
<div className="border-t border-b border-teal-800/40 bg-teal-950/20 py-3 px-6 text-center mb-8">
  <p className="text-xs text-teal-400">
    Shell page — full research guide coming soon
  </p>
</div>
```

---

## Supabase Seed (6 fields)

After creating data.ts and page.tsx, seed the minimum fields to Supabase:

```sql
INSERT INTO spots (
  slug,
  name,
  country,
  region,
  coordinates,
  wind_season,
  skill_level_range,
  disciplines,
  page_status
) VALUES (
  'slug-here',
  'Spot Name',
  'Country',
  'Region',
  ST_GeomFromText('POINT(lng lat)', 4326),
  'May–October',
  'Intermediate–Advanced',
  ARRAY['Freeride', 'Wave'],
  'shell'
);
```

`page_status` field: `'shell'` | `'full'` | `'partial'`

---

## Admin Page Update

After creating a shell page, update `app/admin/spots/page.tsx`:

1. Find the spot in the `pending` array
2. Move it to a new `wip` array (create WIP tier if it doesn't exist yet)
3. Add `status: 'wip'` to the spot entry

### WIP tier in admin UI

The WIP tier appears between Partial and Pending in the admin page. Label: "WIP Shells". Shows: spot name, country, region, date created, link to live shell page.

---

## Batch Mode

When processing multiple shell pages in sequence:

1. Read full pending list from `app/admin/spots/page.tsx`
2. For each spot:
   a. Check if `app/spots/<slug>/` directory exists
   b. Pull coordinates from gmaps CSV or run Playwright lookup
   c. AI-infer wind season, skill level, disciplines from spot name + country
   d. Create data.ts (shell exports only)
   e. Create page.tsx (shell layout)
   f. Supabase seed
   g. Move spot from pending → wip in admin page
3. Report: "Shell created: [spot name] — [slug]" after each

Aim: ~2–3 minutes per spot at shell tier.
