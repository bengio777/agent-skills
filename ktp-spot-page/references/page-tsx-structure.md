# page.tsx Structure Guide

## Required Section Order

Every spot page must include all 13 sections in this order:

```
1.  Hero                    id="hero"           — stats pills + tagline
2.  Mini Map                (embedded in hero)  — immediately after title/badge, before description
3.  Kite Spots              id="kite-spots"     — named spot cards
4.  Wind & Conditions       id="conditions"     — monthly table + kite size guide
5.  Camps & Schools         id="camps"          — all camps listed (not just bookable)
6.  Culture & Landscape     id="culture"        — geography, people, history, language
7.  Community & Pro Scene   id="community"      — pros, competitions, records
8.  Activities              id="activities"     — beyond the kite (rest days)
9.  Food & Social Scene     id="food"           — dishes, restaurants, vibe
10. Transport & Logistics   id="logistics"      — airport, visa, currency, getting around
11. Safety                  id="safety"         — NEW REQUIRED SECTION
12. Book Your Trip          id="book"           — camp cards + fly + protect
13. KTP Edge                id="differentiation"— what nobody else tells you
```

Dev-only (conditional on `process.env.NODE_ENV === 'development'`):
- Verified Facts panel
- HITL Gaps panel
- Unverified Flags panel

---

## Mini Map (placement rule — never violate)

The mini map appears **immediately after the title + skill-badge header, before the description** in every kite spot card. Never at the bottom.

Card order: `title + badge` → `mini map` → `shortDescription` → disciplines → hazards → access

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
      <div className="absolute inset-0 flex items-center justify-center"
           style={{ background: "rgba(9,9,11,0.6)", backdropFilter: "blur(2px)" }}>
        <p className="text-xs text-zinc-400 italic">Coordinates pending: local verification required</p>
      </div>
    )}
  </div>
)}
```

---

## Safety Section (NEW — required)

```tsx
<section id="safety" className="py-20">
  <SectionLabel>Safety</SectionLabel>
  <SectionHeading>Know Before You Go</SectionHeading>

  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
    {/* Hazard Summary */}
    <div className="rounded-xl p-6 border border-zinc-800" style={{ background: "#111111" }}>
      <p className="text-xs font-semibold tracking-widest uppercase text-amber-400 mb-1">Hazards</p>
      <p className="text-sm text-zinc-400 leading-relaxed">{safetyInfo.hazardSummary}</p>
    </div>

    {/* Rescue Services */}
    <div className="rounded-xl p-6 border border-zinc-800" style={{ background: "#111111" }}>
      <p className="text-xs font-semibold tracking-widest uppercase text-amber-400 mb-1">Rescue</p>
      <p className="text-sm text-zinc-400 leading-relaxed">{safetyInfo.rescueServices}</p>
    </div>

    {/* Travel Advisory */}
    <div className="rounded-xl p-6 border border-zinc-800" style={{ background: "#111111" }}>
      <p className="text-xs font-semibold tracking-widest uppercase text-amber-400 mb-1">Travel Advisory</p>
      <p className="text-sm text-zinc-400 leading-relaxed">{safetyInfo.travelAdvisory}</p>
    </div>

    {/* Medical */}
    <div className="rounded-xl p-6 border border-zinc-800" style={{ background: "#111111" }}>
      <p className="text-xs font-semibold tracking-widest uppercase text-amber-400 mb-1">Medical</p>
      <p className="text-sm text-zinc-400 leading-relaxed">{safetyInfo.medicalFacilities}</p>
    </div>
  </div>
</section>
```

---

## Book Your Trip Section

### Structure

```
Stay (camp card grid — full width md:col-span-3)
  └── Camp cards (filter by bookingUrl)
  └── Browse all links (Booking.com, TripAdvisor, Airbnb — destination-filtered)
Fly + Protect (2-col row — md:col-span-3)
  └── Fly block (airport IATA, routing, kite bag policy, Aviasales link)
  └── Protect block (key risks, World Nomads link)
```

### Camp card pattern

**Wrapper must be `<div>` — never `<a>`.** Multiple CTAs (Book + Maps) = invalid nested anchors.

```tsx
<div className="flex flex-col rounded-lg overflow-hidden border border-zinc-700 bg-zinc-800/60 hover:border-teal-600/50 hover:bg-zinc-800 transition-colors group">

  {/* Photo */}
  {camp.photoUrl && (
    <div className="relative h-32 w-full overflow-hidden bg-zinc-700">
      <img src={camp.photoUrl} alt={camp.name}
           className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300" />
    </div>
  )}

  <div className="flex flex-col justify-between flex-1 p-4">
    <div>
      {/* Type + Dry badges */}
      <div className="flex flex-wrap gap-1.5 mb-2">
        <span className="text-xs px-2 py-0.5 rounded-full bg-zinc-700 text-zinc-300 capitalize">{camp.type}</span>
        {!camp.alcohol && (
          <span className="text-xs px-2 py-0.5 rounded-full bg-amber-950/60 text-amber-400 border border-amber-800/30">Dry</span>
        )}
      </div>

      {/* Name */}
      <p className="text-sm font-semibold text-white mb-1 leading-snug">{camp.name}</p>

      {/* Gear + priceLevel */}
      <p className="text-xs text-teal-400 mb-2">
        {camp.gearBrand}
        {camp.priceLevel && <span className="text-zinc-500 ml-1">{'$'.repeat(camp.priceLevel)}</span>}
      </p>

      {/* Rating row */}
      {(camp.rating !== undefined || camp.reviewCount !== undefined) && (
        <p className="text-xs text-zinc-400 mb-2">
          {camp.rating !== undefined && <span className="text-amber-400">★ {camp.rating}</span>}
          {camp.reviewCount !== undefined && <span> · {camp.reviewCount.toLocaleString()} reviews</span>}
        </p>
      )}

      {/* Review snippet */}
      {camp.reviewSnippet && (
        <p className="text-xs text-zinc-500 italic line-clamp-2 mb-3 leading-relaxed">{camp.reviewSnippet}</p>
      )}
    </div>

    <div>
      {/* Price + Book */}
      <div className="flex items-center justify-between mb-2">
        <span className="text-xs text-zinc-400">
          {camp.nightlyRate
            ? <><span className="text-white font-semibold">{camp.nightlyRate}</span><span className="text-zinc-500">/night</span></>
            : camp.priceRange}
        </span>
        <a href={camp.bookingUrl} target="_blank" rel="noopener noreferrer"
           className="text-xs text-teal-400 hover:text-teal-300 font-medium transition-colors">
          Book →
        </a>
      </div>

      {/* Maps link */}
      {camp.googleMapsUrl && (
        <a href={camp.googleMapsUrl} target="_blank" rel="noopener noreferrer"
           className="text-xs text-zinc-500 hover:text-zinc-300 transition-colors flex items-center gap-1">
          <span>📍</span> View on Maps →
        </a>
      )}
    </div>
  </div>
</div>
```

### Browse all links

Destination-specific search URLs:
- Booking.com: `https://www.booking.com/searchresults.html?ss=DESTINATION&lang=en-us`
- TripAdvisor: `https://www.tripadvisor.com/Hotels-g{PLACE_ID}-{DESTINATION}-Hotels.html`
- Airbnb: `https://www.airbnb.com/s/DESTINATION/homes`

### Fly block content

Always include:
- Nearest airport + IATA code (bold)
- Routing (direct vs connecting, via which hub)
- Kite bag policy for the primary airline (if known)
- Aviasales affiliate link: `https://www.aviasales.com/?marker=506969`

### Protect block content

Always include:
- The single biggest risk specific to this destination (not generic "travel safely")
- World Nomads link: `https://www.worldnomads.com/`

---

## Shell Banner (Shell mode only)

Insert immediately below the hero section:

```tsx
<div className="border-t border-b border-teal-800/40 bg-teal-950/20 py-3 px-6 text-center">
  <p className="text-xs text-teal-400">
    Shell page — full research guide coming soon
  </p>
</div>
```

---

## Section header pattern

```tsx
<SectionLabel>Label Text</SectionLabel>
<SectionHeading>Heading Text</SectionHeading>
```

`SectionLabel` renders as small uppercase tracking-widest teal text.
`SectionHeading` renders as large bold white text.
Both are defined as local components in page.tsx.

---

## Divider pattern

```tsx
<Divider />
```

Used between every major section. Defined as a local component.

---

## TypeScript import pattern

```tsx
import {
  spotMeta,
  heroStats,
  kiteSpots,
  camps,
  monthlyWindData,
  kiteSizeGuide,
  activities,
  signatureDishes,
  restaurants,
  logisticsBlocks,
  safetyInfo,      // new
  travelInfo,      // new
  ktpDifferentiation,
  verifiedFacts,
  hitlGaps,
  unverifiedFlags,
  type SkillLevel,
} from "./data";
```
