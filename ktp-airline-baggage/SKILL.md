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

## 5. Schema v2.2 — DB Enum Reference

These are the only valid enum values. Normalization from raw extraction to these enums
happens in `seed_supabase.py` via `ROUTE_REGION_MAP` and `FEE_APPLIES_PER_MAP`.
**Never change the extraction prompt to output these directly** — normalize at seed time.

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

**Key normalization rules:**
- `free_bags_included` defaults to `0` if null (NOT NULL constraint)
- `fee_applies_per`: `"sector"` → `"segment"` (Virgin Australia pattern — add to `FEE_APPLIES_PER_MAP`)
- When seeding fails with an enum error → add new mapping to `ROUTE_REGION_MAP`, don't change the extraction prompt

**Verify after seeding:**
```sql
SELECT airline_iata, COUNT(*) FROM airline_bag_fees GROUP BY airline_iata ORDER BY airline_iata;
```

---

## 6. Layer History & Coverage

| Layer | Airlines | Count | Notes |
|-------|----------|-------|-------|
| 1 | UA, LH, FR | 3 | Pilot layer |
| 2 | EK, TK, LA, KQ, AT, U2, AK, QF, TP, CM | 10 | Global hubs |
| 3 | QR, JL, NH, KE, EY | 5 | Asia/Middle East — all non-headless |
| 4 | AA, IB, SQ, ET, A3, VY | 6 | Americas + Europe expansion |
| 5 | DL, AC, KL, BA, VS, AF, LX, OS, AZ, SK, AY, DY, W6, EW, DE, PC, XQ, UX, NT, WN, B6, WS, F9, NK, AM, Y4, AV, G3, AD, JA, H2, TS, BW, FZ, G9, WY, MS, MK, TC, FA, JQ, VA, NZ, HA, FJ, PR, 5J, GA, VJ, AI | 50 | Major global expansion |
| 6 | TR, ID, MM, 7C, MH, TN, 6E, SG, GF, WB, SA, TO, V7, LS, BY, DM, AR, G4, SY | 19 | Southeast Asia + Africa + budget Europe — **next layer** |

**Total coverage: 74 airlines across 6 layers** (Layer 6 not yet scraped).

**Alliance map** (for `ALLIANCE_MAP` in `process_canonical.py`):
- Star Alliance: LX, OS, AY, AC, AV, MS, AI, SQ, NH, LH, TK, TP, UA, ET
- SkyTeam: DL, AF, KL, AM, AZ, UX, G3, SK, KE, AT, KQ, LA
- Oneworld: BA, IB, AK, QF, AA, JL, QR, EY, A3

---

## 7. Known Data Gaps

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

## 8. Adding a New Layer — Checklist

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

## 9. Autonomous Agent Mode

The baggage pipeline is designed to run as a fully autonomous agent — no human in the loop
required for routine refreshes and layer expansion.

### When to run autonomously
- Weekly cron: refresh all airlines where `last_verified` is > 7 days old
- On-demand: when a new layer is added to `LAYER_MAP`
- Triggered: when a policy URL returns NOT_FOUND (airline redesigned their site)

### Agent execution loop

```
FOR each airline in coverage_list WHERE last_verified > 7 days OR status = pending:
  1. verify_urls.py --iata XX
     - OK → proceed to scrape
     - BOT_BLOCK → scrape with --no-headless
     - NOT_FOUND → research new URL via web search, update policy_urls, retry
     - TIMEOUT/ERROR → log as gap, skip this run
  2. scrape_policies.py --iata XX [--no-headless if BOT_BLOCK]
     - Check data/raw/{iata}-*.json is non-empty
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

### Running the agent headlessly
```bash
claude -p "Run the KTP airline baggage agent. Working directory: /Users/benjamingiordano/Projects/kite-the-planet. Run the full autonomous pipeline loop for all airlines where last_verified > 7 days. Use the ktp-airline-baggage skill for rules. Report coverage summary when done."
```

### What the agent should NOT do autonomously
- Delete or overwrite existing canonical files without a `--force` flag
- Change DB enum definitions in `seed_supabase.py` without flagging for human review
- Add more than one new layer in a single run (avoid uncontrolled expansion)
- Modify `PIPELINE.md` (this is a human-reviewed source of truth)

---

## 10. Secrets & Environment

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
