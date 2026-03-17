---
name: media-restructure
description: Use when building or updating the Apple Photos nested album structure from a geo block map, running media_restructure.py, adding new travel destinations to blocks.json, or running incremental imports after returning from a trip.
---

# Media Restructure

Operational skill for `media_restructure.py` in `~/Projects/photos-geo-organizer`. Builds a nested Apple Photos album structure driven by a geo block map. Fully additive — never removes photos or albums.

**Block map:** `~/Projects/kite-the-planet/tools/blocks.json`
**Full SOP:** `docs/agents/MediaLibraryAgent.md` in the repo.

---

## Workflow

### First run or after adding new blocks
```
python3 ~/Projects/photos-geo-organizer/media_restructure.py --block-map ~/Projects/kite-the-planet/tools/blocks.json --dry-run
```
Review photo counts per block. Then execute:
```
python3 ~/Projects/photos-geo-organizer/media_restructure.py --block-map ~/Projects/kite-the-planet/tools/blocks.json --yes
```

### After importing new photos (incremental)
```
python3 ~/Projects/photos-geo-organizer/media_restructure.py --block-map ~/Projects/kite-the-planet/tools/blocks.json --incremental --yes
```
Reads `restructure_state.json` to process only photos imported since last run. Falls back to full scan if state file missing.

### Single destination reprocess
```
python3 ~/Projects/photos-geo-organizer/media_restructure.py --block-map ~/Projects/kite-the-planet/tools/blocks.json --block "Dakhla" --yes
```

### Sync favorites
```
python3 ~/Projects/photos-geo-organizer/media_restructure.py --block-map ~/Projects/kite-the-planet/tools/blocks.json --sync-favorites --yes
```

---

## Album Structure

```
[Year Folder]
  └── [Location] [Date Range]
        ├── Master              ← all matched photos
        ├── Sony Mirrorless     ← if content
        ├── DJI                 ← if content
        ├── Videos              ← if content
        └── Favorites           ← if content, via --sync-favorites
```

---

## Assignment Priority

| Priority | Source | Destination |
|---|---|---|
| 1 | GPS within block centroid + radius | Master |
| 2 | No-GPS device override (Sony/DJI date range) | Master + device sub-album |
| 3 | Date-range fallback (unambiguous single block) | Master + Unlocated/Review |
| 4 | No match | Unlocated/Review only |
| 5 | Favorited flag | Also added to Favorites sub-album |

---

## Block Map Structure

Located at `~/Projects/kite-the-planet/tools/blocks.json`. Each block:

```json
{
  "location": "Dakhla",
  "album": "Dakhla | Aug 2025",
  "year": "2025",
  "start_date": "2025-08-05",
  "end_date": "2025-08-26",
  "centroid_lat": 23.71,
  "centroid_lon": -15.94,
  "radius_km": 50
}
```

Use `no_gps_overrides` to map Sony/DJI device-specific date ranges to blocks when GPS is missing.

**Adding a new destination:** Copy an existing block, update all fields. Run `media_discovery.py` first to get GPS clusters and device date ranges for the new trip.

---

## CLI Flags

| Flag | Description |
|---|---|
| `--block-map PATH` | Path to blocks.json (required) |
| `--dry-run` | Preview assignments without writing to Photos |
| `--yes` / `-y` | Execute |
| `--min-year YEAR` | Only process assets from this year onwards |
| `--block NAME` | Only process blocks matching this location name |
| `--incremental` | Only process photos imported since last run |
| `--sync-favorites` | Sync favorited photos into Favorites sub-albums |

---

## Key Operational Notes

- Run directly in terminal — not as background process
- Expect 15–30 min for full library scan on large library
- Do not interact with Photos while it runs
- Safe to re-run: Photos deduplicates natively, script is additive only
- `restructure_state.json` saved next to block map after each successful run

---

## Common Issues

| Symptom | Fix |
|---|---|
| Photos running slow / hanging | Run in foreground terminal, not background |
| New block not picking up photos | Check date range overlap with adjacent blocks — overlapping blocks cause no-match for dates in the overlap |
| Sony/DJI photos not assigned | Add `no_gps_overrides` entry with device-specific date range for that destination |
| Unlocated/Review growing unexpectedly | Run `media_discovery.py` to identify which photos lack GPS and which blocks they fall into |
