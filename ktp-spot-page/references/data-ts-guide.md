# data.ts Build Guide

## Overview

Every spot page exports 13 typed data objects from `app/spots/<slug>/data.ts`.
This file maps each section of the research package to its data.ts export.
Read the research package from `docs/research/<slug>-research-package.md` end-to-end before starting.

---

## Export Map: Research Package → data.ts

| Research Package Section | data.ts Export | Required? |
|--------------------------|----------------|-----------|
| Spot header (name, country, tagline) | `spotMeta` | Yes |
| Hero stats pills | `heroStats` | Yes |
| Named Kite Spots | `KiteSpot` interface + `kiteSpots[]` | Yes |
| Camps and Accommodation | `Camp` interface + `camps[]` | Yes |
| Wind → Monthly Breakdown | `monthlyWindData[]` | Yes |
| Wind → Kite Size Guide | `kiteSizeGuide[]` | Yes |
| Tourist Activities | `activities[]` | Yes |
| Food → Signature Dishes | `signatureDishes[]` | Yes |
| Food → Named Restaurants/Bars | `restaurants[]` | Yes |
| Transport and Logistics | `logisticsBlocks` (keyed object) | Yes |
| KTP Differentiation Opportunities | `ktpDifferentiation[]` | Yes |
| Safety zone (hazards, rescue, travel advisory) | `safetyInfo` | Yes — new standard |
| Travel-specific zone (visa, currency, language) | `travelInfo` | Yes — new standard |
| Verified Facts | `verifiedFacts[]` | Yes (dev panel) |
| Human-in-the-Loop Gaps | `hitlGaps[]` | Yes (dev panel) |
| Unverified / Flagged | `unverifiedFlags[]` | Yes (dev panel) |

---

## spotMeta

```ts
export const spotMeta = {
  name: string;           // "Dakhla" — kite community name, not city name
  country: string;
  region: string;         // "Western Sahara, Southern Morocco"
  flag: string;           // emoji flag "🇲🇦"
  tagline: string;        // 2–4 sentence hook — no generic adjectives
  coordinates: string;    // "-23.71° N, 15.93° W" format for display
  coastType: string;      // specific: water type, depth, wind angle
  ikoPresence: string;    // named IKO schools present
  windDays: string;       // "~300 wind days per year (April–October season)"
};
```

---

## heroStats

```ts
export const heroStats = [
  { label: "Wind Season", value: "Apr–Oct" },
  { label: "Water Temp (peak)", value: "22–24°C" },
  { label: "Peak Wind", value: "20–30 kts" },
  { label: "Peak Months", value: "Jun–Aug" },
];
```

Always 4 entries. Values come from Monthly Breakdown section.

---

## KiteSpot interface + kiteSpots[]

```ts
export type SkillLevel =
  | "All Levels" | "Beginner" | "Intermediate" | "Advanced"
  | "Intermediate+" | "Intermediate–Advanced";

export interface KiteSpot {
  name: string;               // community name, not address
  shortDescription: string;   // 3–5 sentences — specific, named, no generics
  skillLevel: SkillLevel;
  disciplines: string[];      // ["Freeride", "Freestyle", "Foil", "Wave", "Wing", "Lessons"]
  hazards: string;            // honest, specific — name the hazard, not just "be careful"
  tideDependent: boolean;
  access: string;             // how to get there, cost, 4x4 requirement, etc.
  coordinates?: string;       // "-23.714689,-15.771376" — run Playwright lookup
  coordinatesPending?: boolean; // true only if genuinely unverifiable
}
```

**Rules:**
- Every named launch zone, wave spot, flatwater area, and downwinder route gets its own entry
- `shortDescription` must name specific features — depth, bottom type, wind angle, who uses it
- `hazards` must be specific — "offshore winds north of the spit" not "use caution"
- Run Playwright coordinate lookup for every spot before setting `coordinatesPending: false`

---

## Camp interface + camps[]

See `references/camp-card-build.md` for the full Camp interface and all field rules.

---

## monthlyWindData[]

```ts
export const monthlyWindData = [
  {
    month: "Jan",
    wind: "15–20 kts",        // range, not single number
    windyDays: "60%",          // consistency percentage
    waterTemp: "22°C",
    notes: string,             // specific — name the season state, dominant conditions
  },
  // ... 12 entries total, one per month
];
```

**Rules:**
- All 12 months required — even off-season (explain why it's off-season)
- `notes` must be specific: "Cyclone season; camp operations suspended" not "off-season"
- Source: Windfinder, Windguru, IKSurfMag, or research package wind section

---

## kiteSizeGuide[]

```ts
export const kiteSizeGuide = [
  {
    season: "Peak season label",
    sizes: "9–12m",
    notes: string,  // specific conditions that drive the size recommendation
  },
];
```

Rider reference: 75–80 kg intermediate. Include 3–5 entries covering peak, shoulder, and off-season.

---

## activities[]

```ts
export const activities = [
  {
    icon: string,           // single emoji
    name: string,           // specific activity name
    category: string,       // "Nature" | "Culture" | "Water Sports" | "Lifestyle" | "Wildlife"
    description: string,    // 2–3 sentences, specific and named
    price: string,          // specific if known, "Contact operator" if not
    requiresVehicle: boolean,
  },
];
```

Minimum 5 activities. Named operators where available.

---

## signatureDishes[] + restaurants[]

```ts
export const signatureDishes = [
  {
    name: string,
    description: string,  // what it is, key ingredients, where to find it
  },
];

export const restaurants = [
  {
    name: string,
    type: string,         // "Beachfront café", "Local market stall", etc.
    vibe: string,         // one-line character note
    mustOrder: string,    // specific dish or drink
    priceRange: string,   // "Budget" | "Mid-range" | "Upscale" or specific range
  },
];
```

Minimum 6 signature dishes, 4 named restaurants.

---

## logisticsBlocks

```ts
export const logisticsBlocks = {
  airport: {
    code: string,       // IATA: "VIL"
    headline: string,   // "Dakhla International Airport"
    notes: string,      // routing, airlines, kite bag policy
  },
  visa: {
    headline: string,
    notes: string,      // requirements by passport, cost, e-visa availability
  },
  currency: {
    headline: string,
    notes: string,      // local currency, ATM availability, card acceptance, cash tips
  },
  getting_around: {
    headline: string,
    notes: string,      // taxi, rental car, transfer costs, 4x4 requirements
  },
  connectivity: {
    headline: string,
    notes: string,      // SIM card options, data costs, camp wifi quality
  },
  safety: {
    headline: string,
    notes: string,      // travel advisory level, specific risks, medical facilities
  },
  wetsuit: {
    headline: string,
    notes: string,      // water temp range year-round, recommended thickness
  },
};
```

All 7 keys required.

---

## safetyInfo (NEW — required in all pages)

```ts
export const safetyInfo = {
  hazardSummary: string,       // overall risk level and most important hazards
  rescueServices: string,      // what rescue infrastructure exists (boat, IKO, none)
  crowdLevel: string,          // how crowded and when
  travelAdvisory: string,      // current advisory level and source (gov.uk, travel.state.gov)
  medicalFacilities: string,   // nearest hospital/clinic with location
  emergencyContacts: string,   // coast guard, local emergency number
};
```

**Source:** research package Safety section, government travel advisory sites.
**If AI-inferred:** add `// AI-INFERRED — verify` comment on the object.

---

## travelInfo (NEW — required in all pages)

```ts
export const travelInfo = {
  visaSummary: string,         // brief: "EU/US passports: visa-free 90 days"
  currency: string,            // "Moroccan Dirham (MAD). ATMs in Dakhla town."
  paymentNorms: string,        // "Cash preferred at camps; cards at hotels"
  language: string,            // official + practical: "Arabic, Hassaniya, French widely spoken"
  bestTravelMonth: string,     // single best month with reason
  tipping: string,             // local norm
};
```

**Source:** Country-level facts — accurate via AI inference without a research package.
**If AI-inferred:** add `// AI-INFERRED — verify` comment on the object.

---

## ktpDifferentiation[]

```ts
export const ktpDifferentiation = [
  {
    angle: string,    // the specific differentiation angle
    insight: string,  // the insight competitors miss — named, specific, verifiable
  },
];
```

6–12 entries. Every entry must name something specific that competitors omit or get wrong.

---

## Dev panel exports (verifiedFacts, hitlGaps, unverifiedFlags)

```ts
export const verifiedFacts: string[] = [
  "Named fact with source. (Source: URL)",
];

export const hitlGaps: string[] = [
  "Specific question only first-hand knowledge can answer",
  // Minimum 8 entries
];

export const unverifiedFlags: string[] = [
  "Claim — why it's flagged",
];
```

These power the dev-only HITL panel rendered conditionally in page.tsx.

---

## Handling missing data

| Situation | Rule |
|-----------|------|
| Research package has no data for a field | Use `""` (empty string) for text, omit optional fields |
| AI-inferred content | Add `// AI-INFERRED — verify` comment on the line or object |
| Conflicting sources | Pick the more conservative value; add to `unverifiedFlags` |
| No research package | AI-infer everything except coordinates (always run Playwright) |
| nightlyRate not found by scraper | Set `nightlyRate: undefined` — never guess |
