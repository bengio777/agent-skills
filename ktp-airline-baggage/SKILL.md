---
name: ktp-airline-baggage
description: >
  KTP airline baggage intelligence pipeline. Fires automatically when editing any file
  in tools/baggage-scraper/ or any file matching **/airline*, **/baggage*, or
  **/airline-baggage*. Encodes the complete 4-step pipeline (verify→scrape→extract→seed),
  bot-block registry per airline IATA, subdomain whitelist procedure, schema v2.2 field
  definitions, normalization rules, layer history, known data gaps, and autonomous agent
  mode. This pipeline is foundational to Trip Builder true cost calculations — every
  airline added improves the accuracy of the core value prop. Use this skill whenever
  touching baggage scraper scripts, adding new airlines, or running the autonomous agent.
metadata:
  filePattern: "tools/baggage-scraper/**,**/airline_bag*,**/baggage-scraper*,**/airline-baggage*"
  priority: 90
---

# KTP Airline Baggage Intelligence

> Canonical references: `tools/baggage-scraper/PIPELINE.md` · `tools/baggage-scraper/scrape_policies.py` · `tools/baggage-scraper/seed_supabase.py`
> Working directory for all commands: `tools/baggage-scraper/`

---

## Why this pipeline exists

The Trip Builder true cost formula is:
```
true_cost_usd = ticket_price + (baggage_one_way_usd × 2)
```

Every airline in the database makes this calculation more accurate. Without baggage data, the Trip Builder falls back to a $65 flat estimate — which is often wrong by $100+. This is KTP's hardest moat to replicate.

---

## 0. Pre-Run Reads (Always First)

Before running any pipeline step or autonomous loop, read these two files:

```bash
cat data/canonical/index.json      # 71 complete, 7 pending with blockers
cat data/verify-cache.json         # last verified date + URL status per airline
```

**index.json** tells you: which airlines are `complete`, which are `pending`, and what the specific blocker is for each pending airline (Imperva, 404, JS-render, manual-only, timeout). Do not attempt re-scraping pending airlines without reading their blocker first.

**verify-cache.json** tells you: when each airline was last verified and the per-URL status. If `verified_date` is within 7 days, skip verify step.

**Known pending blockers (as of Layer 5):**

| IATA | Airline | Blocker |
|------|---------|---------|
| LA | LATAM | Imperva bot block — all scrape attempts empty |
| AT | Royal Air Maroc | 404 — policy URL needs research |
| AK | AirAsia | JS-rendered tables — extraction incomplete |
| G9 | Air Arabia | PDF-only fees — manual write required |
| MK | Air Mauritius | JS-expanded sections — manual write required |
| WY | Oman Air | Phone-only fees — manual write required |
| TC | Air Tanzania | 35s timeout — site may be unreachable |

---

## 1. The 4-Step Pipeline

Each step is a **hard gate**. Fix problems before advancing. Never skip a step.

```
Step 1: verify_urls.py     → classify every URL as OK / BOT_BLOCK / REDIRECT / NOT_FOUND / TIMEOUT / ERROR
Step 2: scrape_policies.py → Playwright scrape → data/raw/{iata}-*.json
Step 3: process_canonical.py → Claude API extraction → data/canonical/{IATA}-baggage-intelligence.json
Step 4: seed_supabase.py   → upsert canonical JSON → airline_bag_fees, airline_sports_fees
```

### Commands

**Step 1 — Verify**
```bash
python verify_urls.py --layer N      # check all airlines in a layer
python verify_urls.py --iata XX      # check one airline
python verify_urls.py --broken       # CI-friendly: only print broken URLs
```

**Step 2 — Scrape**
```bash
python scrape_policies.py --iata XX              # single airline, headless (default)
python scrape_policies.py --iata XX --no-headless  # for BOT_BLOCK airlines
python scrape_policies.py --layer N              # all airlines in a layer
python scrape_policies.py --iata XX --wait 5     # add 5s wait for stubborn bot blocks
```

**Step 3 — Extract**
```bash
python process_canonical.py --layer N    # all Layer N airlines missing canonical files
python process_canonical.py --iata XX   # one airline
python process_canonical.py             # all airlines missing canonical files
python process_canonical.py --force     # overwrite existing
python process_canonical.py --dry-run   # print prompt, no API call
```
Requires `ANTHROPIC_API_KEY` in `tools/.env`.

**Step 4 — Seed**
```bash
python seed_supabase.py                  # seed all canonical files
python seed_supabase.py --iata XX        # seed one airline
python seed_supabase.py --dry-run        # print rows without inserting
python seed_supabase.py --skip-profiles  # skip kit_weight_profiles
```
Requires `SUPABASE_URL` + `SUPABASE_SERVICE_KEY` in `tools/.env`.

---

## 2. URL Status Reference

| Status | Meaning | Action |
|--------|---------|--------|
| OK | 200 response | Ready to scrape |
| BOT_BLOCK | 403/429/503 | Run `--no-headless`; if still empty add `--wait 5` |
| REDIRECT | Moved | Check final URL; update policy_url if domain changed |
| NOT_FOUND | 404/410 | Find real URL before scraping — hard stop |
| TIMEOUT | No response | Retry once; then flag for manual |
| ERROR | DNS/connection | Domain may be wrong or down |

---

## 3. Bot-Block Registry

### Radware Bot Manager — requires `--no-headless` + load fallback
```
LX (Swiss), OS (Austrian), AY (Finnair), EW (Eurowings), DE (Condor), XQ (SunExpress)
```
These redirect to `validate.perfdrive.com` — that's a bot challenge, not a whitelist gap.

### Generic bot detection — `--no-headless` sufficient
```
DL (Delta), BA (British Airways), VS (Virgin Atlantic), AF (Air France),
AM (Aeromexico), AV (Avianca), PR (Philippine Airlines), GA (Garuda Indonesia)
```

### If still empty after `--no-headless --wait 5`
Classify as `MANUAL` in `index.json` — see Known Data Gaps section.

---

## 4. Subdomain Whitelist

Before scraping any new layer, check that Playwright is allowed to navigate to the
final redirect URL. The verifier checks this automatically — look for `⚠ WHITELIST GAP`
warnings in `verify_urls.py` output.

**How to add a subdomain:**
Add to `.claude/settings.json` → `allowedTools`:
```json
"mcp__playwright__browser_navigate(url: \"https://support.southwest.com/*\")"
```

**Layer 5 subdomains (already required):**
- `support.southwest.com` (WN)
- `help.jetblue.com` (B6)
- `faq.flyfrontier.com` (F9)
- `customersupport.spirit.com` (NK)
- `ayuda.avianca.com` (AV)
- `cda.skyairline.com` (H2)
- `help.cebupacificair.com` (5J)

Always run `verify_urls.py --broken` after adding subdomains to confirm the gap is closed.

---

## 5. Schema v2.2 — Complete Field Reference

### Canonical JSON structure

```json
{
  "meta": {
    "airline_iata": "UA",
    "airline_name": "United Airlines",
    "loyalty_program": "MileagePlus",
    "alliance": "Star Alliance",
    "source_url": "https://www.united.com/en/us/fly/baggage.html",
    "last_verified": "2026-03-27",
    "source_quality": "official",
    "notes": "Optional free-text notes"
  },
  "airline_bag_fees": [ ... ],
  "airline_sports_fees": [ ... ],
  "kit_weight_profiles": [ ... ]
}
```

### airline_bag_fees fields (BAG_FEE_FIELDS)

| Field | Type | Notes |
|-------|------|-------|
| `airline_iata` | str | 2-3 char IATA code |
| `airline_name` | str | Full airline name |
| `loyalty_program` | str | Program name or null |
| `loyalty_tier` | str | e.g. "standard", "silver", "gold", "elite" |
| `alliance_tier_required` | str | e.g. "Star Alliance Gold" or null |
| `fare_class` | str | e.g. "economy", "business", "first" |
| `route_region` | str | DB enum (see below) |
| `allowance_concept` | str | "piece" or "weight" |
| `free_bags_included` | int | Defaults to 0 — NOT NULL |
| `weight_limit_kg` | float | Per-bag weight limit |
| `size_limit_cm` | str | e.g. "158cm linear" |
| `fee_bag_1` | float | First checked bag fee |
| `fee_bag_2` | float | Second checked bag fee |
| `fee_bag_3plus` | float | Third+ bag fee |
| `fee_currency` | str | ISO currency code |
| `fee_applies_per` | str | "segment" or "journey" |
| `advance_discount_amount` | float | Online booking discount |
| `source_quality` | str | "official", "inferred", "aggregator" |
| `source_url` | str | Page URL |
| `last_verified` | str | ISO date |
| `notes` | str | Free-text |

### airline_sports_fees fields (SPORTS_FEE_FIELDS)

| Field | Type | Notes |
|-------|------|-------|
| `airline_iata` | str | |
| `bag_type` | str | DB enum (see below) |
| `route_region` | str | DB enum |
| `loyalty_tier` | str | |
| `alliance_tier_required` | str | |
| `is_free` | bool | True if no surcharge |
| `is_additive_to_checked_bag_fee` | bool | True if paid on top of normal bag fee |
| `surcharge_amount` | float | |
| `surcharge_currency` | str | |
| `oversize_fee_amount` | float | |
| `oversize_fee_waived` | bool | |
| `overweight_fee_amount` | float | |
| `overweight_fee_per_kg` | float | |
| `weight_limit_kg` | float | |
| `size_limit_cm` | str | |
| `max_weight_kg` | float | |
| `max_length_cm` | float | |
| `fee_applies_per` | str | "segment" or "journey" |
| `advance_booking_required` | bool | |
| `advance_booking_hours` | int | |
| `advance_discount_amount` | float | |
| `promotion_expires` | str | ISO date |
| `alternate_sport_categories` | str | |
| `alternate_surcharge_amount` | float | |
| `alternate_surcharge_currency` | str | |
| `alternate_route_region` | str | |
| `customer_coaching` | str | Key advice for kite travelers |
| `alliance_waiver_applies` | bool | |
| `applies_to_iata_exceptions` | str | |
| `exception_description` | str | |
| `source_quality` | str | |
| `source_url` | str | |
| `last_verified` | str | |
| `notes` | str | |

### DB Enum Reference

| Column | Valid values |
|--------|-------------|
| `route_region` | `intra-domestic`, `intra-europe`, `transatlantic`, `transpacific`, `us-latin-america`, `middle-east-africa`, `intercontinental` |
| `fee_applies_per` | `segment`, `journey` |
| `bag_type` (sports) | `board_bag`, `ski_snowboard`, `golf`, `bicycle`, `diving`, `other_sports` |
| `source_quality` | `official`, `inferred`, `aggregator` |

**Tables and composite keys:**
- `airline_bag_fees` → key: `(airline_iata, route_region, loyalty_tier, fare_class)`
- `airline_sports_fees` → key: `(airline_iata, bag_type, route_region, loyalty_tier)`
- `kit_weight_profiles` → key: `profile_name`

---

## 6. Normalization Rules

### ROUTE_REGION_MAP (complete — 27 entries as of Layer 5)

When seeding fails with an enum error, add to this map in `seed_supabase.py`. **Never change the extraction prompt to output DB enums directly** — normalize at seed time.

```python
ROUTE_REGION_MAP = {
    "international":               "intercontinental",
    "intra-regional":              "intercontinental",
    "global":                      "intercontinental",
    "intra-asia":                  "intercontinental",
    "japan-europe":                "intercontinental",
    "long-haul-caribbean-africa-asia": "intercontinental",
    "long-haul-indian-ocean":      "intercontinental",
    "long-haul-other":             "intercontinental",
    "intra-pacific":               "transpacific",
    "south-pacific":               "transpacific",
    "transatlantic-main":          "transatlantic",
    "north-america":               "intra-domestic",
    "intra-us-canada":             "intra-domestic",
    "intra-hawaii":                "intra-domestic",
    "intra-turkey":                "intra-domestic",
    "intra-latam":                 "us-latin-america",
    "latin-caribbean":             "us-latin-america",
    "north-america-latam":         "us-latin-america",
    "intra-africa":                "middle-east-africa",
    "middle-east":                 "middle-east-africa",
    "africa":                      "middle-east-africa",
    "intra-middle-east":           "middle-east-africa",
    "asia-me":                     "middle-east-africa",
    "intra-european":              "intra-europe",
    "intra-spain-balearics":       "intra-europe",
    "canary-islands-europe-africa": "intra-europe",
    "canary-islands-europe-me":    "intra-europe",
}
```

### FEE_APPLIES_PER_MAP

```python
FEE_APPLIES_PER_MAP = {
    "sector": "segment",   # Virgin Australia pattern
    # "flight" is a likely future variant — add if encountered
}
```

### Other normalization rules
- `free_bags_included` defaults to `0` if null (NOT NULL constraint)
- When seeding fails with an enum error → add to `ROUTE_REGION_MAP`, don't change extraction prompt

**Verify after seeding:**
```sql
SELECT airline_iata, COUNT(*) FROM airline_bag_fees GROUP BY airline_iata ORDER BY airline_iata;
```

---

## 7. Alliance Waiver Patterns

Sports equipment fees are waived for elite alliance members on many airlines. The extraction prompt handles this but the agent should validate alliance waiver data against these patterns:

| Alliance Tier | Applies To | What's Waived |
|---------------|-----------|---------------|
| Oneworld Emerald | AA, BA, IB, JL, QF, QR, EY, A3 | First sports bag free |
| Star Alliance Gold | LH, UA, TK, NH, LX, OS, AY, SQ, ET, AC, TP, AI | Sports fees waived |
| SkyTeam Elite Plus | DL, AF, KL, AM, AZ, SK, KE, AT, KQ, LA, UX, G3 | Sports fees waived |

When `alliance_waiver_applies: true` is set, `alliance_tier_required` should be populated with the specific tier string (e.g., `"Star Alliance Gold"`).

**Alliance map** (for `ALLIANCE_MAP` in `process_canonical.py`):
- Star Alliance: LX, OS, AY, AC, AV, MS, AI, SQ, NH, LH, TK, TP, UA, ET
- SkyTeam: DL, AF, KL, AM, AZ, UX, G3, SK, KE, AT, KQ, LA
- Oneworld: BA, IB, AK, QF, AA, JL, QR, EY, A3

---

## 8. Layer History & Coverage

| Layer | Airlines | Count | Notes |
|-------|----------|-------|-------|
| 1 | UA, LH, FR | 3 | Pilot layer |
| 2 | EK, TK, LA, KQ, AT, U2, AK, QF, TP, CM | 10 | Global hubs |
| 3 | QR, JL, NH, KE, EY | 5 | Asia/Middle East — all non-headless |
| 4 | AA, IB, SQ, ET, A3, VY | 6 | Americas + Europe expansion |
| 5 | DL, AC, KL, BA, VS, AF, LX, OS, AZ, SK, AY, DY, W6, EW, DE, PC, XQ, UX, NT, WN, B6, WS, F9, NK, AM, Y4, AV, G3, AD, JA, H2, TS, BW, FZ, G9, WY, MS, MK, TC, FA, JQ, VA, NZ, HA, FJ, PR, 5J, GA, VJ, AI | 50 | Major global expansion |
| 6 | TR, ID, MM, 7C, MH, TN, 6E, SG, GF, WB, SA, TO, V7, LS, BY, DM, AR, G4, SY | 19 | Southeast Asia + Africa + budget Europe — **next layer** |

**Total coverage: 74 airlines across 6 layers** (Layer 6 not yet scraped).

---

## 9. Known Data Gaps

| IATA | Airline | Fix | Reason |
|------|---------|-----|--------|
| BW | Caribbean Airlines | RE-SCRAPE | Hash-based SPA — run `--no-headless` with `#/baggage` anchor |
| TS | Air Transat | RE-SCRAPE | React-rendered tables — JS execution + networkidle needed |
| G9 | Air Arabia | MANUAL | Redirects to cabin FAQ only; checked bag fees in PDF |
| MK | Air Mauritius | MANUAL | JS-expanded sections; fee structure unclear |
| WY | Oman Air | MANUAL | Excess baggage fees not published online; requires phone contact |
| TC | Air Tanzania | MANUAL | 35s timeout; site may be unreachable |

**Re-scrape procedure (BW, TS):**
```bash
python scrape_policies.py --iata BW --no-headless
# Verify data/raw/bw-*.json has non-empty tables
python process_canonical.py --iata BW
python seed_supabase.py --iata BW
```

**Manual procedure (G9, MK, WY, TC):**
1. Find fee data via PDF, phone, or aggregator
2. Write `data/canonical/{IATA}-baggage-intelligence.json` manually
3. Set `source_quality: "inferred"` or `"aggregator"` on all rows
4. Run `python seed_supabase.py --iata XX`

---

## 10. Scraper Implementation Details

Understanding these helps diagnose empty raw files:

- **Page wait strategy**: networkidle first; falls back to `load` if networkidle times out (common on SPA sites)
- **Scrape timeout**: 35 seconds per airline (raised from default for slow sites)
- **Post-load wait**: 6 seconds after page load for JS-heavy sites
- **Delay between URLs**: 1500ms between each URL in an airline's `policy_urls` list
- **Bot evasion**: `--disable-blink-features=AutomationControlled` flag applied to all Playwright launches
- **Empty file detection**: after scraping, check `data/raw/{iata}-*.json` file size — files under 200 bytes are effectively empty

**Diagnosing empty raw files:**
1. File exists but is tiny → page loaded but tables didn't render → try `--no-headless` then `--wait 5`
2. No file created → scraper threw an error → check terminal output for traceback
3. File has content but extraction produces 0 rows → JS-rendered content not captured → classify MANUAL

**split_canonical.py warning**: This is a one-time migration utility that splits legacy combined canonical files into per-airline files. Do NOT run it again — it would corrupt current data by re-splitting already-correct files.

---

## 11. Adding a New Layer — Checklist

1. Draft airline entries in `scrape_policies.py` with best-guess `policy_urls`
2. Add layer to `LAYER_MAP` in `verify_urls.py` (keep in sync)
3. Run `verify_urls.py --layer N` — fix every NOT_FOUND before proceeding
4. Check subdomain whitelist — add any new subdomains to `.claude/settings.json`
5. Run `scrape_policies.py --layer N` — use `--no-headless` for BOT_BLOCK airlines
6. Verify `data/raw/` — empty files = needs re-scrape with `--wait 5` or mark MANUAL
7. Run `process_canonical.py --layer N` (API) or multi-window mode (free — see below)
8. Spot-check 5–10 canonical files for reasonable row counts
9. Run `seed_supabase.py` — fix enum errors by adding to `ROUTE_REGION_MAP`
10. Update `data/canonical/index.json` pending array for any 0-row airlines
11. Append to PROGRESS.md: row counts, bot-blocks, gaps, new normalization entries

**Multi-window mode (free, no API key) — 5 windows, 10 airlines each:**
Paste this prompt into each Claude Code window:
```
Working directory: tools/baggage-scraper
For each airline: read data/raw/{iata}-*.json → extract → write data/canonical/{IATA}-baggage-intelligence.json
Use schema v2.2 (see process_canonical.py EXTRACTION_PROMPT). Key rule: use descriptive
route_region values from source — do NOT normalize to DB enums here; normalization happens in seed_supabase.py.

Window 1: [10 IATA codes]
```
Layer 5 window allocation: W1: DL AC KL BA VS AF LX OS AZ SK · W2: AY DY W6 EW DE PC XQ UX NT WN · W3: B6 WS F9 NK AM Y4 AV G3 AD JA · W4: H2 TS BW FZ G9 WY MS MK TC FA · W5: JQ VA NZ HA FJ PR 5J GA VJ AI

---

## 12. Autonomous Agent Mode

The baggage pipeline is designed to run as a fully autonomous agent — no human in the loop
required for routine refreshes and layer expansion.

### When to run autonomously
- Weekly cron: refresh all airlines where `last_verified` is > 7 days old
- On-demand: when a new layer is added to `LAYER_MAP`
- Triggered: when a policy URL returns NOT_FOUND (airline redesigned their site)

### Agent execution loop

```
BEFORE LOOP:
  - Read data/canonical/index.json — note pending airlines and their blockers
  - Read data/verify-cache.json — note last_verified dates per airline

FOR each airline in coverage_list WHERE last_verified > 7 days OR status = pending:
  - IF airline is in known MANUAL blockers (G9, MK, WY, TC) → skip, log
  1. verify_urls.py --iata XX
     - OK → proceed to scrape
     - BOT_BLOCK → scrape with --no-headless
     - NOT_FOUND → research new URL via web search, update policy_urls, retry
     - TIMEOUT/ERROR → log as gap, skip this run
  2. scrape_policies.py --iata XX [--no-headless if BOT_BLOCK]
     - Check data/raw/{iata}-*.json is non-empty (>200 bytes)
     - If empty → retry with --wait 5; if still empty → classify MANUAL
  3. process_canonical.py --iata XX
     - Write data/canonical/{IATA}-baggage-intelligence.json
  4. seed_supabase.py --iata XX
     - Catch enum errors → add to ROUTE_REGION_MAP → retry once
     - Verify row count > 0 after seeding
  5. Log: airline, rows_inserted, gaps, timestamp → PROGRESS.md
END

AFTER LOOP:
  - Run verify_urls.py --broken → report any new broken URLs
  - Check coverage_list against layer_map for unscraped airlines → flag as expansion candidates
  - Output coverage report: total airlines, row counts, gaps, next recommended layer
```

### Expansion behavior
When the agent identifies unscraped airlines in `LAYER_MAP`, it should:
1. Research the airline's baggage policy URL (web search or known patterns)
2. Add to `scrape_policies.py` AIRLINES list using the exact format below
3. Add the IATA to `LAYER_MAP` in `verify_urls.py` under the correct layer
4. Run the full 4-step pipeline for the new airline
5. Report back: airline added, rows inserted, any issues

### Airline entry format (`scrape_policies.py` AIRLINES list)

```python
# --- Layer N: [layer description] ---
{
    "iata": "XX",                        # 2-3 char IATA code, uppercase
    "name": "Full Airline Name",         # Official name as used on their website
    "loyalty_program": "Program Name",   # Set to None if no loyalty program
    "policy_urls": [
        "https://www.airline.com/en/baggage/checked-baggage",   # checked bag fees page
        "https://www.airline.com/en/baggage/sports-equipment",  # sports/kite equipment page
        # Add more URLs if fees are split across multiple pages
    ],
},
```

**URL discovery rules:**
- Always include a checked baggage fee page AND a sports equipment page when both exist
- Common URL patterns: `/baggage/`, `/travel-info/baggage`, `/en/baggage-fees`, `/help/baggage`
- For help-center airlines (Ryanair, JetBlue, Southwest): use `help.airline.com` or `support.airline.com` URLs
- Run `verify_urls.py --iata XX` immediately after adding — fix NOT_FOUND before any scraping
- If the sports equipment URL doesn't exist, omit it (don't add a broken URL)

**`loyalty_program` values by common pattern:**
- Major US carriers: MileagePlus (UA), AAdvantage (AA), SkyMiles (DL), TrueBlue (B6), Rapid Rewards (WN)
- Set to `None` for budget/LCC carriers with no loyalty program (FR, W6, F9, NK, etc.)

**After adding any airline entry:**
- Add IATA to `LAYER_MAP` in `verify_urls.py` under the correct layer number
- Run `verify_urls.py --iata XX` to confirm URL is reachable before scraping

### Per-airline inline comments
`scrape_policies.py` has inline comments on every airline documenting: kite spot relevance,
piece vs weight allowance concept, alliance tier, known bot behavior, and special URL notes.
Read these comments before scraping or researching an airline — they encode field-tested knowledge
from prior scrape sessions.

### Running the agent headlessly
```bash
claude -p "Run the KTP airline baggage agent. Working directory: /Users/benjamingiordano/Projects/kite-the-planet. Run the full autonomous pipeline loop for all airlines where last_verified > 7 days. Use the ktp-airline-baggage skill for rules. Report coverage summary when done."
```

### What the agent should NOT do autonomously
- Delete or overwrite existing canonical files without a `--force` flag
- Change DB enum definitions in `seed_supabase.py` without flagging for human review
- Add more than one new layer in a single run (avoid uncontrolled expansion)
- Modify `PIPELINE.md` (this is a human-reviewed source of truth)
- Attempt MANUAL-only airlines (G9, MK, WY, TC) — these require human data collection

---

## 13. Secrets & Environment

Both layers of secrets required:
```bash
tools/.env          # shared: ANTHROPIC_API_KEY, SUPABASE_URL, SUPABASE_SERVICE_KEY
tools/baggage-scraper/.env  # tool-specific additions if any
```

Loading pattern in scripts:
```python
load_dotenv(Path(__file__).resolve().parent.parent / ".env")  # shared tools/.env
load_dotenv()  # tool-specific (additive)
```

**Python package requirements:**
```
anthropic>=0.86.0
playwright>=1.44.0
python-dotenv>=1.0.0
supabase>=2.0.0
```
