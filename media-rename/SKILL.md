---
name: media-rename
description: Use when renaming Insta360, DJI, or Sony camera files to the standard geo-naming convention. Invoke at the start of any media rename session to load folder structure, naming rules, location resolution logic, and country-specific conventions before touching any files.
---

# Media Rename

Operational skill for `media_rename.py` in `~/Projects/photos-geo-organizer`. Covers the full rename workflow and all naming conventions established in production.

**Full SOP:** `docs/agents/MediaLibraryAgent.md` in the repo.

---

## Workflow

1. Dry-run first — always review before executing
2. Check for edge cases — device detection, spot stripping, generic subfolders
3. Execute
4. Verify — check file count matches, `rename_log.json` created

**Standard folder:**
```
python3 ~/Projects/photos-geo-organizer/media_rename.py --input "/path/to/Country" --dry-run --recursive
```

**Nepal (requires country-specific block map):**
```
python3 ~/Projects/photos-geo-organizer/media_rename.py --input "/path/to/Nepal" --block-map ~/Projects/photos-geo-organizer/nepal_rename_blocks.json --dry-run --recursive
```

Replace `--dry-run` with `--yes` to execute.

**Path note:** The Insta360 base path uses a Unicode apostrophe (U+2019) in "Benjamin's". Use the Python exec invocation pattern (passing sys.argv manually) to avoid shell escaping issues with this path.

---

## Naming Format

```
[Location]_[YYYY-MM-DD]_[Device]_[Duration]_[Notes]_[Original].ext
```

- Duration: video only (mp4, mov, insv). Omitted for stills.
- Notes: verbatim, only when present.

---

## Location Naming Rules

| Situation | Location prefix |
|---|---|
| Root-level folder only | `Vietnam`, `Aus` |
| Named subfolder | `Country-Spot` (e.g. `Brazil-Jeri`) |
| Generic subfolder (Timelapse, edited) | Inherit parent spot (e.g. `Vietnam-HaGiangLoop`) |
| Subfolder name equals country name | Fall back to root level |
| Trailing digits in subfolder name | Strip them (`Rasutsu2` becomes `Rasutsu`) |

**Location resolution priority:** GPS > date (block map) > folder name. Date from block map beats folder name without conflict.

---

## Country Folder Status

| Country | Convention | Block map needed? |
|---|---|---|
| Australia (Aus) | Root level, short name OK | No |
| Brazil | Brazil-Jeri, Brazil-Atins | No |
| Japan | Japan-Niseko, Japan-Teine, Japan-Rasutsu | No |
| Madagascar | Madagascar-Emerald-Sea, Madagascar-Motorbike | No |
| Nepal | Nepal-Pokhara, Nepal-Lower-Mustang | Yes — nepal_rename_blocks.json |
| Thailand | Root level | No |
| Vietnam | Vietnam-HaGiangLoop, Vietnam-Kiting | No |

All above folders are already renamed as of 2026-03-16.

---

## Notes Stripping Rules

- Strip first notes token if it is a 2-4 uppercase legacy code (FL, MOR, DAK)
- Strip spot name token if it equals the location hint with optional trailing digits only
- Keep tokens like `TouringDay1` — non-digit suffix means meaningful content, not redundant spot name

---

## Device Detection

Reads the copyright atom from MP4. If the value is not in the known set (DJI-Action4, DJI-Drone, Insta360, Sony), falls back to folder name, then parent folder name, then Unknown.

---

## Common Issues

| Symptom | Fix |
|---|---|
| 0 files found despite files existing | Path has Unicode apostrophe — use Python exec invocation pattern |
| Device shows as version string like 60.3.100 | Known devices filter rejects it; folder fallback applies automatically |
| Nepal files get wrong sub-region or conflict warning | Use nepal_rename_blocks.json, not the main blocks.json |
| Generic subfolder gets wrong location prefix | Folder name must be in GENERIC_SUBFOLDERS set in the script |
