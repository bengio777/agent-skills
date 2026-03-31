---
name: ktp-research-dispatch
description: >
  KTP parallel research dispatcher for kite spot pages. Use this skill whenever starting
  a new spot page build, asked to research a kite spot, or running batch research across
  multiple spots. Dispatches parallel subagents — one per research category — and returns
  a structured markdown package mapped directly to data.ts fields, ready for ktp-spot-page
  to consume without additional research. This is the unlock for scaling spot pages from
  49 to 198. Fire it before or at the start of any ktp-spot-page session where a data.ts
  does not already exist. Single-spot mode for deep pages; batch mode for stubs.
metadata:
  priority: 75
---

# KTP Research Dispatch

> Canonical output target: `app/spots/{slug}/data.ts`
> Canonical schema reference: `app/spots/dakhla/data.ts`
> Companion skill: `ktp-spot-builder` (rules + quality standards)
> Companion skill: `ktp-spot-page` (build workflow)

---

## What This Produces

A **research package** — structured markdown covering all data.ts fields — delivered as
a single artifact ready for ktp-spot-page to assemble into a live spot page. No follow-up
research needed after a complete package is returned.

The package covers these data.ts exports:
- `spotMeta` + `heroStats`
- `kiteSpots[]` with coordinates
- `camps[]` with booking URLs, ratings, nightly rates
- `monthlyWindData[]` + `kiteSizeGuide[]`
- `activities[]` + `signatureDishes[]` + `restaurants[]`
- `logisticsBlocks` (airport, visa, money, sim, transport, safety)
- `ktpDifferentiation[]`
- `verifiedFacts[]`

---

## Modes

### Single-Spot Mode (deep research)
One spot, full package. 5 parallel subagents, ~25–40 web searches total.
Use when: building a complete spot page.

**Invocation:**
```
Research [Spot Name], [Country] for a KTP kite spot page.
Run the ktp-research-dispatch skill. Single-spot mode.
Return a complete research package.
```

### Batch Mode (stubs)
Multiple spots, breadth over depth. 3–5 searches per spot, parallel.
Use when: populating stub pages, triaging spots, or pre-filling data for 10+ spots at once.

**Invocation:**
```
Research the following kite spots for KTP stub pages:
[Spot A], [Spot B], [Spot C]...
Run the ktp-research-dispatch skill. Batch mode — 3 searches per spot, heroStats + kiteSpots only.
Return one package per spot.
```

---

## Parallel Dispatch — 5 Subagents

Spawn all 5 in the same turn. Do not run sequentially.

Each subagent has a single category. Pass the spot name and country. Merge outputs when all complete.

---

### Agent 1 — Wind Intelligence

**Responsible for:** `heroStats`, `monthlyWindData[]`, `kiteSizeGuide[]`

**Searches to run (pick 3–5):**
- `[Spot] kitesurfing wind statistics monthly`
- `[Spot] Windfinder average wind speed`
- `[Spot] best kite season month`
- `[Spot] wind direction prevailing`
- `[Spot] kite size guide`

**Sources to prioritize:**
1. Windfinder.com (monthly averages, wind rose)
2. Windguru.cz (historical data)
3. Local kite camp websites (often publish wind stats)
4. IKO school listings (sometimes include season info)
5. Kitesurfing forum trip reports (real-world kite size reports)

**Output fields:**
```
heroStats:
  Wind Days/Year: [N]+
  Avg Wind Speed: [N] kts
  Water Temp: [min–max°C / min–max°F]
  Peak Season: [Month–Month]

monthlyWindData: (12 rows)
  month | wind (kts range) | windyDays (%) | waterTemp (°C / °F) | notes

kiteSizeGuide: (4–5 rows)
  season | recommended sizes | notes
```

---

### Agent 2 — Spot Geography

**Responsible for:** `kiteSpots[]`, `spotMeta` (coordinates, region, tagline)

**Searches to run (pick 4–6):**
- `[Spot] kitesurfing spots map`
- `[Spot] kite beach GPS coordinates`
- `[Spot] kiteboarding launch points`
- `[Spot] [specific named spot] kite conditions`
- `[Spot] IKO kite school locations`
- `[Spot] kite spot beginner intermediate advanced`

**Sources to prioritize:**
1. IKO school directory (iko.org) — spot locations + school density
2. Google Maps — individual beach/spot names + coordinates
3. Kitesurf.co.uk / Kiteforum.com / iKitesurf — trip reports
4. Camp websites — often name their specific access beaches
5. YouTube kite videos — riders name spots in descriptions

**Output fields per kite spot:**
```
name: [official local name]
shortDescription: [2–3 sentences — conditions, who it suits, timing notes]
skillLevel: [Beginner | Intermediate | Advanced | All Levels | Intermediate+ | Intermediate–Advanced]
disciplines: [array: Freestyle | Freeride | Foil | Wave | Surf | Beginners | Speed]
hazards: [specific, named — offshore wind / rocks / crowd / tidal / boat traffic]
tideDependent: [true | false]
access: [how to get there — walk, 4x4, boat, distance from camps]
coordinates: [lat,lng — from Google Maps @lat,lng URL]
coordinatesPending: [true only if genuinely unverifiable]
```

**Coordinates rule:** Extract `@lat,lng` from Google Maps URL. Never guess.
Flag `coordinatesPending: true` only for unnamed offshore breaks or genuinely inaccessible spots.

---

### Agent 3 — Accommodations

**Responsible for:** `camps[]`

**Searches to run (pick 4–5):**
- `[Spot] kite camp accommodation`
- `[Spot] kitesurfing resort booking`
- `[Spot] kite school hotel [country]`
- `[Spot] ION Club [or other known operators]`
- `[Spot] kite camp reviews tripadvisor`

**Sources to prioritize:**
1. Booking.com — nightly rates, ratings, review count, photos
2. TripAdvisor — reviews, rankings, `reviewSnippet` candidates
3. Camp/resort own websites — gearBrand, characterNote, alcohol policy
4. Google Maps — ratings, address, coordinates
5. IKO directory — certified schools

**Output fields per camp:**
```
name: [official name]
type: [lagoon | wave | luxury]
characterNote: [1–2 sentences — vibe, what makes it distinct, any notable quirks]
gearBrand: [F-ONE | North | Cabrinha | Naish | Mixed | ION Club | etc.]
priceRange: [budget string e.g. "~€80/night" or "Mid-range" or "Premium"]
highlight: [single most compelling thing about this camp]
alcohol: [true | false — important for Muslim travelers]
bookingUrl: [Booking.com search URL or direct URL]
googleMapsUrl: [Google Maps place URL]
rating: [N.N from Google or TripAdvisor]
reviewCount: [N]
address: [street address from Google Maps]
nightlyRate: ["$NNN" — from Booking.com or Google]
reviewSnippet: [optional — one punchy direct quote from a real review]
```

---

### Agent 4 — Travel Logistics

**Responsible for:** `logisticsBlocks` (airport, visa, money, sim, transport, safety)

**Searches to run (pick 4–5):**
- `[Spot] nearest airport IATA code flights`
- `[Spot country] visa requirements [major nationalities]`
- `[Spot country] currency exchange tips`
- `[Spot] SIM card local operator`
- `[Spot] kite gear airline baggage policy`

**Sources to prioritize:**
1. Official airport websites — IATA code, routes
2. Government visa portals (iata.org/travel) — visa requirements
3. Lonely Planet / travel forums — practical money/SIM tips
4. Airline websites / Seat61.com — routes to the airport
5. Local kite camp FAQs — transport logistics between airport and camps

**Output fields:**
```
airport:
  code: [IATA]
  name: [full airport name]
  distance: [km from city/spot]
  routes: [array of "City (CODE) — Airline, frequency"]
  kiteGear: [any airline with notable baggage policy for kite gear]
  note: [optional practical tip]

visa:
  visaFree: [nationalities + duration]
  requirements: [passport validity, onward travel etc.]
  warning: [any specific entry concern]

money:
  currency: [Name (CODE)]
  warning: [closed currency, ATM availability, etc.]
  atm: [practical ATM notes]
  tip: [one specific money-saving/practical tip]
  cards: [where cards work vs cash-only]

sim:
  recommended: [operator name]
  reason: [why — coverage area]
  avoid: [operator to skip + reason]
  price: [SIM cost, data cost]
  esim: [eSIM options if available]

transport:
  [key: value pairs for camp shuttle, car rental, local taxis, etc.]

safety:
  overall: [one sentence]
  city: [city-specific notes]
  waterSafety: [kite-specific water safety notes]
  avoid: [areas or situations to avoid]
```

---

### Agent 5 — KTP Differentiation + Culture

**Responsible for:** `ktpDifferentiation[]`, `activities[]`, `signatureDishes[]`, `restaurants[]`, `verifiedFacts[]`

**Searches to run (pick 5–7):**
- `[Spot] kitesurfing unique experience`
- `[Spot] things to do non-kite activities`
- `[Spot] local food specialties dishes`
- `[Spot] best restaurants`
- `[Spot] cultural experiences travel`
- `[Spot] kite competition events`
- `[Spot] [country] kite travel tips reddit forum`

**Sources to prioritize:**
1. Kite camp blog posts / FAQs — differentiation angles, local culture
2. Kitesurfing forum trip reports — real traveler perspectives
3. TripAdvisor / Google Maps — activities, restaurants
4. Tourism board sites — cultural activities, events
5. Reddit travel/kite threads — honest real-world tips

**ktpDifferentiation output** (3–5 entries, each with):
```
angle: [a specific, named KTP framing — not generic]
pullQuote: [1–2 sentences written in KTP voice — specific, surprising, vivid]
context: [1 sentence: why this angle is better than what competitors say]
```

Quality bar: each angle must be verifiable and specific. Flag any that use generic adjectives (amazing, stunning, perfect). See ktp-spot-builder for copy quality rules.

**activities output** (5–10 entries):
```
name | category | description (2–3 sentences) | icon | price | requiresVehicle
```

**signatureDishes output** (4–8 dishes):
```
name | description (1–2 sentences — the why, not just the what)
```

**restaurants output** (3–6 restaurants):
```
name | type | coordinates | note (what makes it worth going)
```

**verifiedFacts output** (5–15 facts):
```
fact: [specific verifiable claim] | source: [publication or website]
```
Only include facts with a named source. Never include AI-inferred facts in verifiedFacts.

---

## Merging the Package

After all 5 agents complete, merge into one structured markdown document:

```markdown
# Research Package: [Spot Name], [Country]
Generated: [date] | Searches: [total] | Sources: [total]

## spotMeta
[key: value pairs]

## heroStats
[table]

## kiteSpots
[one section per spot]

## camps
[one section per camp]

## monthlyWindData
[table]

## kiteSizeGuide
[table]

## activities
[list]

## signatureDishes
[list]

## restaurants
[list]

## logisticsBlocks
[airport / visa / money / sim / transport / safety]

## ktpDifferentiation
[3–5 entries]

## verifiedFacts
[list with sources]

## Research Gaps
[Any fields that could not be filled — flag with reason]
```

---

## Quality Gate (Before Handing Off)

Before passing the package to ktp-spot-page, verify:

- [ ] At least 3 named kite spots with coordinates (not just "the beach")
- [ ] At least 2 camps with booking URLs and nightly rates
- [ ] 12 months of wind data (even if estimated)
- [ ] Airport IATA code confirmed
- [ ] Visa requirements for EU/UK/US travelers confirmed
- [ ] 3 ktpDifferentiation entries — none use generic adjectives
- [ ] verifiedFacts have named sources
- [ ] All temperatures in dual-unit format (°C / °F) — see ktp-spot-builder rule

Flag any missing fields in a **Research Gaps** section. Do not silently omit — gaps are
visible so ktp-spot-page knows what to manually fill.

---

## Handoff to ktp-spot-page

Once the package passes the quality gate, hand off with:

```
Research package for [Spot] is complete. [N] kite spots, [N] camps, [N] gaps flagged.
Ready for ktp-spot-page. Start with the shell or go straight to data.ts?
```

The ktp-spot-page skill takes it from there. Do not start writing data.ts until the
user confirms the handoff — they may want to review gaps first.

---

## Batch Mode — Reduced Scope

For batch research (10+ spots), run only Agents 1, 2, and 4 per spot. Skip Agents 3 and 5.
Output per spot:
- `heroStats` (4 values)
- `kiteSpots[]` (2–3 spots minimum, coordinates required)
- `logisticsBlocks.airport` only

This is enough to populate stub pages and unlock trip builder functionality.
Full packages are built per-spot in subsequent single-spot sessions.
