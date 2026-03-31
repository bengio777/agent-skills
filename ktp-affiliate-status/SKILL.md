---
name: ktp-affiliate-status
description: >
  KTP affiliate and integration status. Use when working on trip builder, booking flows,
  revenue features, affiliate links, or any file in lib/api/, app/api/trip-builder/,
  or app/dev/trip/. Single source of truth — never ask Ben to re-brief on affiliate status.
metadata:
  filePattern: "app/api/trip-builder/**,lib/api/**,app/dev/trip/**,*affiliate*,*booking*,*revenue*"
  priority: 85
---

# KTP Affiliate & Integration Status

> Last updated: 2026-03-30
> This is the authoritative record. Do not ask Ben what's live — read this.

---

## Live (earning now)

### SafetyWing Ambassador
- **Product:** Nomad Insurance
- **Commission:** ~10% per confirmed sale
- **Affiliate link format:** `https://safetywing.com/nomad-insurance?referenceID=26502172`
- **Dashboard:** hello.safetywing.com
- **Status:** Active — not yet wired into Trip Builder UI

### Travelpayouts
- **Products:** Flights (cheapest tickets matrix)
- **Token:** Active and configured in Vercel env
- **Marker:** In `TRAVELPAYOUTS_MARKER` env var
- **Affiliate link builder:** `buildAffiliateLink()` in `lib/api/travelpayouts.ts`
- **Coverage:** Best for EU/non-US origins. Thin for US/CA/AU (use SerpAPI there)
- **Status:** Integrated in `/api/trip-builder/flights` — smoke test not yet confirmed

---

## Pending approval

### Skyscanner (Impact.com)
- **Account name:** Benjamin Giordano (personal — via @ben_f_gio Instagram, 1,000+ followers)
- **Platform:** Impact.com
- **Applied:** 2026-03-30
- **Status:** In Review — awaiting email notification
- **Action when approved:** Add deep links to flight results for US-origin users

### Hostelworld (Partnerize)
- **Platform:** Partnerize — console.partnerize.com
- **Account:** kitetheplanet
- **Status:** Pending review (shows "requested" in Partnerize UI — not rejected)
- **Contact:** affiliates@hostelworld.com
- **Action when approved:** Add accommodation cards to Trip Builder destination results

---

## Blocked (dependency: CJ activation)

### CJ (Commission Junction)
- **Publisher CID:** 7916093
- **Blocker:** Account Information step stuck at 5/6 — CJ platform API 500 bug on save
- **Workaround:** Retry Edit → Save on the Account Information page; or contact support.cj.com
- **Downstream:** World Nomads AND Booking.com both require CJ to activate first

### World Nomads
- **Network:** CJ
- **Blocked until:** CJ account activates

### Booking.com
- **Network:** CJ
- **Blocked until:** CJ account activates

---

## Dead — do not pursue

### Tequila / Kiwi.com
- **Status:** Platform retired. No replacement public API available.
- **Confirmed:** 2026-03-30
- **Action:** Remove from any task lists or integration plans

---

## Integration architecture

### Flight search routing
```
Origin country in SERPAPI_ORIGIN_COUNTRIES (US, CA, AU, NZ, JP, KR, SG)?
  YES → SerpAPI (Google Flights) — lib/api/serpapi.ts
  NO  → Travelpayouts cheapest tickets — lib/api/travelpayouts.ts
Both run in parallel; Travelpayouts result preferred when available
```

### True cost formula
```
true_cost_usd = ticket_price + (baggage_one_way_usd × 2)
```
Baggage fallback chain: exact route region → worldwide → $65 estimate

### Affiliate link construction
- **Travelpayouts:** `buildAffiliateLink(rawLink)` in `lib/api/travelpayouts.ts`
- **SafetyWing:** `https://safetywing.com/nomad-insurance?referenceID=26502172`
- **Skyscanner:** pending approval — no link yet
- **Hostelworld:** pending approval — no link yet

---

## Trip Builder — revenue surface checklist

When building or modifying Trip Builder results:
- [ ] Flight card: Travelpayouts affiliate link fires via `buildAffiliateLink()`
- [ ] Insurance card: SafetyWing link present (referenceID=26502172) — **not yet built**
- [ ] Accommodation card: Hostelworld link — **blocked, pending approval**
- [ ] True cost calculation includes round-trip baggage fee
- [ ] SerpAPI results for US/CA/AU origins have no booking link (free tier limitation)
