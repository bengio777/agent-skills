# Camp Card Build Guide

## Camp Interface (canonical)

```ts
export interface Camp {
  // Required
  name: string;
  type: "lagoon" | "wave" | "luxury" | "beach" | "adventure";
  characterNote: string;   // 3–5 sentences — specific vibe, named details
  gearBrand: string;       // "North, Duotone (rental fleet)" or "IKO-certified (brand not confirmed)"
  priceRange: string;      // fallback if nightlyRate unavailable
  highlight: string;       // one-line what makes this camp unique vs others
  alcohol: boolean;        // false ONLY for confirmed dry camps; default true

  // Booking
  bookingUrl?: string;     // direct camp website URL; only camps with this appear in card grid

  // Media
  photoUrl?: string;       // "/camps/<slug>.<ext>" — local path only

  // Google Maps enrichment (from scraper)
  googleMapsUrl?: string;
  rating?: number;         // e.g. 4.7
  reviewCount?: number;    // e.g. 312
  address?: string;        // as shown on Google Maps listing
  priceLevel?: number;     // 1–4 ($–$$$$); rarely populated for accommodation
  nightlyRate?: string;    // "$176" — from Google Maps live hotel pricing widget ONLY
  reviewSnippet?: string;  // one pull-quote from reviews; rarely populated via scraper
}
```

**type values by destination:**
- Dakhla: `"lagoon" | "wave" | "luxury"`
- Sakalava Bay: `"luxury" | "beach" | "adventure"`
- New spots: define types that reflect the actual camp character at that destination

---

## Scraper Workflow

### Step 1 — Edit the camps array

Open `tools/camp-scraper/scrape_google_maps.js` and replace the `camps` array:

```js
const camps = [
  { name: 'Camp Name', search: 'Camp Name Destination Country' },
  // one entry per camp — make search specific enough to find the right listing
];
```

### Step 2 — Run

```
node tools/camp-scraper/scrape_google_maps.js
```

Output: console log for each camp + ready-to-paste data.ts snippets + full JSON.

**Wait time:** 8 seconds per camp for hotel price widget to load. 7 camps ≈ ~2 min.

### Step 3 — Paste output into data.ts

Copy the snippet block for each camp. Fields that returned null are omitted — do not add them.

### Step 4 — Restore the camps array

Reset `tools/camp-scraper/scrape_google_maps.js` back to the Dakhla camps array after the run.

---

## Photo Sourcing (dual-source rule)

### Priority order

1. **Booking.com property page** — scrape `max1024x768` signed URLs directly from the property page (not search results). These are high-quality and properly licensed for referral display.

2. **Camp website** — fetch the best hero/gallery image via `node -e` with Referer header. Look for `og:image`, large `<img>` tags, or network response interception for Wix/hosted sites.

3. **Google Maps** — metadata only (rating, reviewCount, nightlyRate, address, googleMapsUrl). Photos from Maps are not downloadable via Playwright.

### Download command pattern

```js
node -e "
const https = require('https');
const fs = require('fs');
const url = 'IMAGE_URL';
https.get(url, { headers: { 'Referer': 'https://www.google.com/maps/' } }, res => {
  res.pipe(fs.createWriteStream('public/camps/SLUG.jpg'));
  res.on('end', () => console.log('done'));
});
"
```

### File naming

- Save to: `public/camps/<slug>.<ext>`
- Serve as: `/camps/<slug>.<ext>` in `photoUrl`
- Slug: lowercase, hyphens, matches camp name (e.g. `dakhla-attitude.jpg`)

### Critical: verify file extension

Always check actual MIME type before finalizing extension:

```bash
file public/camps/<slug>.jpg
```

If output shows `Web/P image data` — rename to `.webp`. Mismatched extensions cause MIME errors and broken images.

```bash
mv public/camps/<slug>.jpg public/camps/<slug>.webp
```

Update `photoUrl` in data.ts to match.

---

## nightlyRate rules

- `nightlyRate` comes **only** from the Google Maps live hotel pricing widget
- The scraper uses `waitForTimeout(8000)` to allow the widget to load — do not reduce this
- If the scraper returns null → set `nightlyRate: undefined` (omit the field)
- **Never guess, approximate, or use priceRange as a substitute**
- If the widget shows a rate but it's obviously wrong (e.g. $1 or $9999) → omit it
- When rendered: `$XX/night` next to the Book button; falls back to `priceRange` text if undefined

---

## Which camps appear in the Book Your Trip grid

Only camps with a `bookingUrl` appear in the card grid:

```tsx
{camps.filter(c => c.bookingUrl).map((camp, i) => (...))}
```

All other camps (those without a direct booking link) appear only in the Camps & Schools section further up the page.

---

## Common issues

| Issue | Fix |
|-------|-----|
| Scraper finds no price | `nightlyRate` stays undefined — that's correct |
| Google Maps shows wrong place | Make the search string more specific (add neighborhood, country) |
| File downloaded as wrong extension | `file` command to check, rename to correct ext, update photoUrl |
| WebP served as .jpg | Browser shows broken image — always rename to .webp |
| Booking.com URL doesn't have signed photos | Use camp website as photo source instead |
| Camp not on Google Maps | Set googleMapsUrl, rating, reviewCount to undefined — skip scraper for this camp |
