---
name: ktp-gmaps-scraper
description: >
  KTP Google Maps operator and school discovery pipeline. Fires automatically when editing
  any file in tools/gmaps-scraper/. Encodes the full multi-script pipeline for discovering
  kite schools and operators, sharded parallel execution, data consolidation, spot-matching,
  and Notion/Supabase output. Use this skill whenever running, debugging, or extending the
  operator database pipeline.
metadata:
  filePattern: "tools/gmaps-scraper/**"
  priority: 85
---

# KTP GMaps Operator Scraper

> Working directory: `tools/gmaps-scraper/`
> Output data: `tools/gmaps-scraper/data/`
> Shared secrets: `tools/.env`

---

## Purpose

Discover and enrich all kite schools, camps, and operators near KTP's 198 spots.
Output feeds the KTP Operators Notion DB and Supabase `operators` table.

**Current scale:** 811 matched schools across known kite spots.

---

## Pipeline Stages

```
Stage 1: Scrape kite schools from Google Maps
  └── scrape_kite_schools.py → data/gmaps-schools.csv
      Queries: "kite school", "kiteboarding school", "kite surf center", etc.
      Covers: 60+ kite destination locations across all major kite countries
      Output fields: name, rating, review_count, address, city, lat, lng,
                     phone, website, google_maps_url, source_query

Stage 2: Scrape IKO-certified schools
  └── scrape_iko.py → data/iko-schools.csv

Stage 3: Consolidate and deduplicate
  └── consolidate_schools.py → data/schools-consolidated.csv
      Merges gmaps + IKO sources, deduplicates by name + coordinates

Stage 4: Match schools to nearest KTP spot
  └── cluster_spots.py → data/schools-matched.csv
      For each school, finds the nearest spot from the KTP spots list
      Assigns nearest_spot, distance_km

Stage 5: Push to Notion Operators DB
  └── build_operators_db.py → KTP Operators Notion DB
      Creates DB if not exists, links each operator to Spot page via Relation
      Filters bad operator names before pushing

Stage 6: Export to Notion Content Pipeline
  └── build_content_pipeline_db.py → KTP Content Pipeline Notion DB

Stage 7 (optional): Enrich with Google Place Details
  └── fetch_place_details.py
      Fetches additional fields: opening hours, price level, photos
      Requires GOOGLE_MAPS_API_KEY
```

---

## Run Commands

```bash
cd /Users/benjamingiordano/Projects/kite-the-planet/tools/gmaps-scraper

# Stage 1: Scrape kite schools (single run)
python scrape_kite_schools.py

# Stage 1: Sharded parallel run (3 workers)
python scrape_kite_schools.py --shard 0 3  # worker 0
python scrape_kite_schools.py --shard 1 3  # worker 1
python scrape_kite_schools.py --shard 2 3  # worker 2
python scrape_kite_schools.py --merge      # combine shards → gmaps-schools.csv

# Stage 2: IKO schools
python scrape_iko.py

# Stages 3–4: Consolidate and match
python consolidate_schools.py
python cluster_spots.py

# Stage 5: Push to Notion
python build_operators_db.py --dry-run    # preview only
python build_operators_db.py              # full push (811 operators)

# Stage 6: Content pipeline DB
python build_content_pipeline_db.py

# Optional: Discover new spots
python discover_new_spots.py
```

---

## Sharded Execution

`scrape_kite_schools.py` supports sharding for parallel runs:
- Divides the LOCATIONS list into N equal shards
- Each worker writes to `data/gmaps-schools-shard-{n}.csv`
- `--merge` combines all shards into `gmaps-schools.csv`

Use sharding when running on multiple terminal windows to speed up the full scrape.
Always run `--merge` before proceeding to Stage 3.

---

## Key Data Files

| File | Contents | Stage |
|------|----------|-------|
| `data/gmaps-schools.csv` | Raw Google Maps results | Output of Stage 1 |
| `data/iko-schools.csv` | IKO certified schools | Output of Stage 2 |
| `data/schools-consolidated.csv` | Deduped merged dataset | Output of Stage 3 |
| `data/schools-matched.csv` | Schools with nearest_spot assigned | Output of Stage 4 |

---

## Environment Variables

| File | Key | Required for |
|------|-----|-------------|
| `tools/.env` | `GOOGLE_MAPS_API_KEY` | Stage 7 (place details enrichment) |
| `tools/.env` | `NOTION_TOKEN` | Stages 5–6 |
| `tools/.env` | `NOTION_SPOTS_DB_ID` | Stage 5 — links operators to spots |
| `tools/.env` | `NOTION_OPERATORS_DB_ID` | Stage 5 — target DB (auto-created if not set) |
| `tools/.env` | `NOTION_PARENT_PAGE_ID` | Stage 5 — fallback parent if DB doesn't exist yet |
| `tools/.env` | `NOTION_CONTENT_DB_ID` | Stage 6 |

---

## Notion Operators DB Schema

Created by `build_operators_db.py`. Matches KTP Operators Notion DB:

| Property | Type | Notes |
|----------|------|-------|
| Name | title | Operator/school name |
| Source | select | gmaps, osm, iko |
| IKO Certified | checkbox | |
| Nearest Spot | relation | Links to KTP Spots DB |
| Distance to Spot (km) | number | |
| Address | rich_text | |
| Website | url | |
| Rating | number | Google Maps rating |
| Country | rich_text | |
| City | rich_text | |
| Lat / Lng | number | |
| Outreach Status | select | Not started → In progress → Responded → Converted |
| Google Maps | url | Direct Google Maps link |
| Date Added | date | |

---

## Bad Operator Name Filter

`build_operators_db.py` filters out low-quality entries before pushing:

```python
BAD_OPERATOR_NAMES = {"results", "", "kite school", "kite surf", "kite center"}
BAD_OPERATOR_PATTERNS = [r"^\[unnamed", r"^osm:", r"^Results$", r"^\s*$"]
```

If new bad-name patterns appear after a scrape, add them here before re-pushing.

---

## Adding New Locations

To scrape a new kite destination, add to the `LOCATIONS` list in `scrape_kite_schools.py`:

```python
("Spot Name Country", "Spot Name Country"),
# e.g.:
("Mui Ne Vietnam", "Mui Ne Vietnam"),
```

Format: `(display_name, search_suffix)`. The search suffix is appended to each query term.
After adding, re-run Stage 1 → Stage 3 → Stage 4 → Stage 5.

---

## Secrets Architecture

```python
load_dotenv(Path(__file__).resolve().parent.parent / ".env")  # shared tools/.env
load_dotenv()  # tool-specific additions (additive)
```

`GOOGLE_MAPS_API_KEY` lives in `tools/.env` — shared with any other tool that needs it.
