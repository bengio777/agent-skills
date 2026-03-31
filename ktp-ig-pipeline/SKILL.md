---
name: ktp-ig-pipeline
description: >
  KTP Instagram intelligence pipeline. Fires automatically when editing any file in
  tools/ig-classifier/. Encodes the full EnsembleData + Claude Haiku classification
  pipeline, Meta app config, token regeneration procedure, error code handling, rate
  limits, checkpoint/resume behavior, and Notion output schema. Use this skill whenever
  running, debugging, or extending the Instagram prospect classifier.
metadata:
  filePattern: "tools/ig-classifier/**"
  priority: 85
---

# KTP Instagram Intelligence Pipeline

> Working directory: `tools/ig-classifier/`
> Meta config reference: `tools/ig-classifier/META-CONFIG.md`
> Shared secrets: `tools/.env` | Tool-specific: `tools/ig-classifier/.env`

---

## Purpose

Classify Ben's 7,441 Instagram following + 1,667 followers as kite influencers, operators,
schools, lodging, or brands. Output feeds the SalesIntelligenceAgent prospect database.

**Scale:** ~9,000 accounts total → ~1,500 kite-relevant (after keyword filter) → full classification run.

---

## Meta App Config

| Key | Value |
|-----|-------|
| Meta App Name | KTP Kite Research |
| Meta App ID | `1214812320844120` |
| Facebook Page | Kite the Planet |
| Facebook Page ID | `1084731161381751` |
| Instagram Account | @ben_f_gio |
| Instagram User ID | `17841400751470586` |

**Required app permissions** (must be "Ready for testing" at `developers.facebook.com/apps/1214812320844120/use_cases/`):
- `instagram_basic`
- `instagram_manage_insights`
- `pages_show_list`
- `pages_read_engagement`

---

## Environment Variables

| File | Key | Notes |
|------|-----|-------|
| `tools/.env` | `ANTHROPIC_API_KEY` | Claude Haiku classification |
| `tools/.env` | `NOTION_TOKEN` | Notion integration |
| `tools/ig-classifier/.env` | `ENSEMBLE_API_TOKEN` | EnsembleData SDK token |
| `tools/ig-classifier/.env` | `NOTION_DATABASE_ID` | KTP Instagram Prospects DB ID |

**Note:** `IG_ACCESS_TOKEN` and `IG_USER_ID` are legacy Meta Graph API keys — no longer used after EnsembleData migration. Do not confuse with `ENSEMBLE_API_TOKEN`.

---

## Run Commands

```bash
cd /Users/benjamingiordano/Projects/kite-the-planet/tools/ig-classifier
source venv/bin/activate

# Smoke test — 5 accounts, ~50 EnsembleData units (exhausts free tier)
python3 smoke_test.py

# Full classification run — 8–12 hours, ~1,500 accounts
python3 classify.py
```

**Prerequisite:** EnsembleData Wood plan ($100/mo) — provides 1,500 units/day → ~150 accounts/day → ~10 days for full run. Free tier (50 units/day) is smoke-test only.

---

## Pipeline Stages

```
Stage 1: Load accounts
  ├── load_following()  → ~/Downloads/instagram-export/.../following.json
  └── load_followers()  → ~/Downloads/instagram-export/.../followers_1.json

Stage 2: Keyword filter
  └── keyword_filter()  → keeps accounts whose username contains kite keywords
      Keywords: kite, kitesurf, kiteboard, surf, wind, pwa, iko, woo,
                school, camp, lodge, resort, hostel, hotel, travel,
                spot, beach, foil, wing

Stage 3: EnsembleData fetch (10 units/call)
  └── fetch_ensemble_profile()
      Endpoint: /instagram/user/detailed-info
      Returns: name, bio, followers_count, media_count, website, username, profile_picture_url
      Rate limit: 200 calls/hour → pipeline sleeps 18s between calls

Stage 4: Claude Haiku classification
  └── CLASSIFICATION_SYSTEM prompt → JSON output
      Categories: influencer | operator | school | lodging | brand | irrelevant
      Fields: category, confidence (High|Medium|Low), reasoning (1 sentence)

Stage 5: Checkpoint + output
  ├── Checkpoints every 50 accounts → checkpoint.csv
  ├── Final CSV output → ktp_prospects.csv
  └── Notion push → KTP Instagram Prospects DB
```

---

## EnsembleData Error Codes

| Code | Error | Action |
|------|-------|--------|
| 491 | Token invalid | Stop run immediately — update `ENSEMBLE_API_TOKEN` |
| 495 | Daily units exhausted | Stop run, checkpoint, exit — resume tomorrow |
| 471/472/474 | Personal/private/restricted account | Classify from username only (no bio data) |
| 473 | Account not found | Skip this account |
| 500 | Server error | Retry once with backoff |

**Personal accounts** (error 471/472/474) do not block the run — classify from username and move on. Only 491 and 495 require stopping.

---

## Token Regeneration (IG_ACCESS_TOKEN — legacy, not currently used)

Tokens last ~60 days. If ever needed again:
1. `developers.facebook.com/tools/explorer`
2. App: **KTP Kite Research** → Token: **Get Page Token** → Page: **Kite the Planet**
3. Add all 4 permissions above → **Generate Access Token**
4. Update `.env`: `IG_ACCESS_TOKEN=<new token>`
5. Verify: `python3 smoke_test.py`

---

## Checkpoint & Resume

- Checkpoints every 50 accounts to `checkpoint.csv`
- If run is interrupted: re-run `python3 classify.py` — it reads `checkpoint.csv` and picks up from last completed account
- Do not delete `checkpoint.csv` between interrupted runs

---

## Output Schema — Notion "KTP Instagram Prospects" DB

| Property | Type | Values |
|----------|------|--------|
| Username | title | Instagram handle |
| Display Name | rich_text | |
| Category | select | influencer, operator, school, lodging, brand, irrelevant |
| Confidence | select | High, Medium, Low |
| Followers | number | |
| Post Count | number | |
| Bio | rich_text | |
| Website | url | |
| Instagram URL | url | |
| Source | select | following, follower |
| Account Type | select | business, personal |
| API Data | select | Yes, No |
| Reasoning | rich_text | Claude classification reasoning |

---

## Classification Categories

| Category | Who |
|----------|-----|
| `influencer` | Individual kite athletes, content creators, enthusiasts with a following |
| `operator` | Kite trip operators, travel agencies, tour companies |
| `school` | Kite schools, IKO/VDWS certified instruction centers |
| `lodging` | Hotels, resorts, hostels, beach camps, accommodation in kite destinations |
| `brand` | Kite gear brands, equipment manufacturers, industry suppliers |
| `irrelevant` | Not related to kitesurfing or kite travel |

---

## Known Pipeline Limitations

- EnsembleData only works for **Business and Creator** accounts — personal accounts return 471/472/474 and are classified from username only
- Keyword filter is username-based only — some relevant accounts with non-keyword usernames will be missed (acceptable for V1)
- Instagram export JSON files must be present at the hardcoded paths in `classify.py`:
  - `~/Downloads/instagram-export/connections/followers_and_following/following.json`
  - `~/Downloads/instagram-export/connections/followers_and_following/followers_1.json`

---

## Secrets Architecture

```python
load_dotenv(Path(__file__).resolve().parent.parent / ".env")  # shared tools/.env
load_dotenv()  # tool-specific tools/ig-classifier/.env (additive)
```

Never put `ENSEMBLE_API_TOKEN` in `tools/.env` — it's ig-classifier-specific.
