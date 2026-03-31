---
name: ktp-supabase-schema
description: >
  KTP Supabase schema reference. Fires automatically when editing migration files, SQL,
  or any file matching **/supabase/** or **/migration*. Encodes all 21 table definitions,
  architecture principles, composite keys, enum values, RLS roles, migration policy, and
  the Sanity/Supabase/API split. Use this skill whenever writing migrations, querying
  Supabase, building features that touch the database, or debugging schema errors.
metadata:
  filePattern: "**/supabase/**,**/*.sql,**/migration*,**/seed*.ts,**/seed*.py"
  priority: 85
---

# KTP Supabase Schema

> Schema spec: `docs/plans/2026-03-10-supabase-schema-design.md`
> Project ID: `gynrvwjmddsycfzivijf` | Region: `eu-west-1`
> Dashboard: `https://supabase.com/dashboard/project/gynrvwjmddsycfzivijf`

---

## Architecture Principles

| Layer | Stores | Join Key |
|-------|--------|----------|
| **Supabase** | Transactional data, user data, scraped data, cached API responses, structured spot data | — |
| **Sanity CMS** | Editorial content: descriptions, guides, rich text, media | `spot.slug` |
| **Live APIs** | Wind/weather, water temp, flight search results | — |

- `spot.slug` is the join key between Supabase spot records and Sanity CMS documents
- **RLS enforced on all tables** — three roles: `admin`, `user`, `operator`
- **Migration policy:** Apply per feature, not all at once. Never run all migrations speculatively.

---

## Migration Policy

Migrations are **not yet run** — schema is spec-complete, awaiting feature-by-feature implementation.

**Rule:** Only apply a migration when the feature that needs it is actively being built.

```bash
# Apply a migration
supabase db push          # from local supabase CLI
# OR via Supabase dashboard SQL editor for single-table changes
```

---

## Table Index

| # | Table | Purpose | Status |
|---|-------|---------|--------|
| 1 | `users` | Core user accounts | Not yet migrated |
| 2 | `user_disciplines` | Discipline + skill level per user | Not yet migrated |
| 3 | `spots` | Core spot records (structured data) | Not yet migrated |
| 4 | `spot_details` | 33-field spot page spec data | Not yet migrated |
| 5 | `spot_travel_advisories` | Cached US State Dept advisory levels | Not yet migrated |
| 6 | `spot_visa_requirements` | Cached VisaDB per nationality | Not yet migrated |
| 7 | `gear_catalog` | 16 manufacturers, current + 5yr history | Not yet migrated |
| 8 | `gear_locker` | User's owned gear | Not yet migrated |
| 9 | `gear_locker_configs` | Named gear configurations | Not yet migrated |
| 10 | `gear_locker_config_items` | Items in each config | Not yet migrated |
| 11 | `trips` | Trip workspace per destination per group | Not yet migrated |
| 12 | `trip_members` | Group members (account + non-account) | Not yet migrated |
| 13 | `trip_gear_bags` | Per-member gear selection per trip | Not yet migrated |
| 14 | `trip_bookings` | Confirmed bookings within a trip | Not yet migrated |
| 15 | `trip_savings` | Savings attribution per component | Not yet migrated |
| 16 | `airline_bag_fees` | Base checked bag fees by tier/route | **Live** (seeded via baggage pipeline) |
| 17 | `airline_sports_fees` | Sports equipment surcharges | **Live** (seeded via baggage pipeline) |
| 18 | `kit_weight_profiles` | Standard kite kit configurations | **Live** (seeded via baggage pipeline) |
| 19 | `operators` | Kite schools, camps, accommodation | Not yet migrated |
| 20 | `prospects` | SalesIntelligenceAgent prospect DB | Not yet migrated |
| 21 | `spot_check_ins` | User visit log | Not yet migrated |

---

## Table Definitions

### `users`
```sql
users
├── id                  uuid          PK, default gen_random_uuid()
├── email               text          unique, not null
├── display_name        text
├── username            text          unique
├── avatar_url          text
├── bio                 text
├── home_beach          text
├── home_airport_iata   text          -- IATA code (e.g. AMS, CDG)
├── country             text
├── preferred_currency  text          default 'USD'
├── preferred_units     enum          metric | imperial
├── years_riding        int
├── instagram_handle    text
├── is_private          boolean       default false
├── role                enum          user | operator | admin
├── onboarding_complete boolean       default false
├── created_at          timestamptz   default now()
└── updated_at          timestamptz   default now()
```

### `user_disciplines`
```sql
user_disciplines
├── id            uuid    PK
├── user_id       uuid    FK → users.id
├── sport         enum    kite | wing
├── discipline    enum    twin_tip | strapless | foil | wing_foil
├── skill_level   enum    no_experience | beginner | intermediate | advanced | high_advanced | professional
├── created_at    timestamptz
└── updated_at    timestamptz
```

### `spots`
```sql
spots
├── id                  uuid    PK
├── name                text    not null       -- "Dakhla"
├── slug                text    unique         -- "dakhla-morocco" — joins to Sanity
├── country             text
├── region              text
├── latitude            float
├── longitude           float
├── sanity_document_id  text                   -- Sanity CMS document ID
├── status              enum    draft | published | archived
├── created_at          timestamptz
└── updated_at          timestamptz
```

### `spot_details`
```sql
spot_details
├── id                      uuid        PK
├── spot_id                 uuid        FK → spots.id, unique
├── spot_type               enum        lagoon | ocean_beach | river_mouth | lake
├── water_types             text[]      -- ['flat', 'wave', 'chop']
├── skill_levels            text[]      -- ['beginner', 'intermediate', 'advanced']
├── peak_season_start       text
├── peak_season_end         text
├── off_season_start        text
├── off_season_end          text
├── wind_direction          enum        onshore | offshore | mix
├── wind_direction_notes    text
├── wind_strength_min_kt    int
├── wind_strength_max_kt    int
├── wind_consistency_pct    int         -- % good wind days peak season
├── is_tide_dependent       boolean     default false
├── foiling_suitable        boolean
├── foiling_notes           text
├── launch_zone_count       int
├── launch_restrictions     text
├── hazards                 text
├── crowd_level             enum        low | moderate | crowded
├── rescue_services         text
├── social_scene_rating     int         -- 1-5
├── social_scene_notes      text
├── affordability_tier      enum        budget | mid | premium
├── anchor_price_notes      text
├── accommodation_types     text[]
├── major_airport_iata      text
├── regional_airport_iata   text
├── local_currency          text
├── payment_norms           text
├── primary_language        text
├── english_spoken          boolean
├── good_for_non_kiters     boolean
├── solo_friendly           boolean
├── best_travel_month       text
├── kite_size_guide         jsonb       -- {"peak": {"min": 9, "max": 12}, "off": {"min": 12, "max": 17}}
└── updated_at              timestamptz
```

### `spot_travel_advisories`
```sql
spot_travel_advisories
├── id              uuid    PK
├── country_code    text    unique    -- ISO 3166-1 alpha-2
├── advisory_level  int               -- 1 (normal) to 4 (do not travel)
├── advisory_title  text
├── advisory_text   text
├── last_updated    timestamptz       -- when State Dept last updated
└── fetched_at      timestamptz       -- when KTP last fetched
```

### `spot_visa_requirements`
```sql
spot_visa_requirements
├── id                  uuid    PK
├── destination_country text    -- ISO country code
├── nationality         text    -- ISO country code
├── visa_required       boolean
├── visa_type           text    -- "eVisa", "visa on arrival", "visa-free"
├── notes               text
└── fetched_at          timestamptz
```

### `gear_catalog`
```sql
gear_catalog
├── id          uuid    PK
├── brand       text    -- "Cabrinha"
├── model       text    -- "Moto"
├── year        int
├── category    enum    kite | board | bar | harness | wetsuit | wing | foil | bag | protection | apparel | accessory
├── subcategory text    -- "12m kite", "twin tip"
├── weight_kg   float
├── length_cm   float
├── width_cm    float
├── height_cm   float
├── bag_type    text    -- "kite bag", "board bag"
├── source_url  text
├── scraped_at  timestamptz
└── created_at  timestamptz
```

### `gear_locker` / `gear_locker_configs` / `gear_locker_config_items`
```sql
gear_locker
├── id               uuid    PK
├── user_id          uuid    FK → users.id
├── gear_catalog_id  uuid    FK → gear_catalog.id
├── nickname         text
├── notes            text
└── created_at       timestamptz

gear_locker_configs
├── id          uuid    PK
├── user_id     uuid    FK → users.id
├── name        text    -- "Dakhla setup"
├── notes       text
└── created_at  timestamptz

gear_locker_config_items
├── id              uuid    PK
├── config_id       uuid    FK → gear_locker_configs.id
└── gear_locker_id  uuid    FK → gear_locker.id
```

### `trips` / `trip_members` / `trip_gear_bags`
```sql
trips
├── id                  uuid    PK
├── organizer_id        uuid    FK → users.id
├── spot_id             uuid    FK → spots.id
├── name                text    -- "Dakhla March 2026"
├── origin_airport_iata text
├── departure_date      date
├── return_date         date
├── status              enum    planning | booked | completed | cancelled
├── created_at          timestamptz
└── updated_at          timestamptz

trip_members
├── id          uuid    PK
├── trip_id     uuid    FK → trips.id
├── user_id     uuid    FK → users.id    nullable (non-account guests)
├── name        text                     -- for non-account guests
├── email       text
├── role        enum    organizer | member
└── joined_at   timestamptz

trip_gear_bags
├── id              uuid    PK
├── trip_id         uuid    FK → trips.id
├── user_id         uuid    FK → users.id
├── gear_locker_id  uuid    FK → gear_locker.id
├── is_bringing     boolean default true
└── created_at      timestamptz
```

### `trip_bookings` / `trip_savings`
```sql
trip_bookings
├── id                  uuid    PK
├── trip_id             uuid    FK → trips.id
├── booking_type        enum    flight | accommodation | car_rental | transfer | insurance
├── provider            text    -- "Hostelworld", "Kiwi.com"
├── booking_reference   text
├── cost_total          float
├── cost_currency       text
├── cost_per_person     float
├── affiliate_link      text
├── modification_policy text
├── status              enum    pending | confirmed | cancelled
├── created_at          timestamptz
└── updated_at          timestamptz

trip_savings
├── id                  uuid    PK
├── trip_id             uuid    FK → trips.id
├── user_id             uuid    FK → users.id    nullable (group-level savings)
├── savings_type        enum    baggage | accommodation | transfer | car_rental
├── estimated_savings   float
├── confirmed_savings   float
├── currency            text
├── baseline_description text   -- "vs. Ryanair €89 + €120 bag fees"
└── created_at          timestamptz
```

### `airline_bag_fees` — LIVE
```sql
airline_bag_fees
├── id                      uuid        PK
├── airline_iata            text
├── airline_name            text
├── loyalty_program         text        nullable
├── loyalty_tier            text        nullable
├── alliance_tier_required  text        nullable    -- "Star Alliance Gold"
├── fare_class              text        nullable
├── route_region            text        CHECK IN ('intra-domestic','intra-europe','transatlantic',
│                                                 'transpacific','us-latin-america',
│                                                 'middle-east-africa','intercontinental')
├── allowance_concept       text        CHECK IN ('piece','weight')
├── free_bags_included      integer     NOT NULL default 0
├── weight_limit_kg         float
├── size_limit_cm           text
├── fee_bag_1               float
├── fee_bag_2               float
├── fee_bag_3plus           float
├── fee_currency            text
├── fee_applies_per         text        CHECK IN ('segment','journey')
├── advance_discount_amount float
├── source_quality          text        CHECK IN ('official','aggregator','inferred')
├── source_url              text
└── last_verified           date
-- Composite key: (airline_iata, route_region, loyalty_tier, fare_class)
```

### `airline_sports_fees` — LIVE
```sql
airline_sports_fees
├── id                          uuid    PK
├── airline_iata                text
├── bag_type                    text    CHECK IN ('board_bag','ski_snowboard','golf',
│                                                 'bicycle','diving','other_sports')
├── route_region                text    -- same enum as airline_bag_fees
├── loyalty_tier                text    nullable
├── alliance_tier_required      text    nullable
├── is_free                     boolean
├── is_additive_to_checked_bag_fee boolean
├── surcharge_amount            float
├── surcharge_currency          text
├── oversize_fee_amount         float
├── oversize_fee_waived         boolean
├── overweight_fee_amount       float
├── overweight_fee_per_kg       float
├── weight_limit_kg             float
├── size_limit_cm               text
├── max_weight_kg               float
├── max_length_cm               float
├── fee_applies_per             text    CHECK IN ('segment','journey')
├── advance_booking_required    boolean default false
├── advance_booking_hours       integer
├── advance_discount_amount     float
├── promotion_expires           date
├── customer_coaching           text    -- plain-language savings tip for UI
├── alliance_waiver_applies     boolean default false
├── source_quality              text
├── source_url                  text
└── last_verified               date
-- Composite key: (airline_iata, bag_type, route_region, loyalty_tier)
```

### `kit_weight_profiles` — LIVE
```sql
kit_weight_profiles
├── id                  uuid    PK
├── profile_name        text    unique    -- "1-kite travel setup"
├── board_bag_kg        float
├── kite_bag_kg         float
├── board_bag_length_cm float
├── board_bag_width_cm  float
├── board_bag_depth_cm  float
└── notes               text
```

### `operators`
```sql
operators
├── id                      uuid    PK
├── user_id                 uuid    FK → users.id    nullable
├── spot_id                 uuid    FK → spots.id
├── name                    text
├── types                   text[]  -- ['school', 'accommodation', 'rental']
├── website                 text
├── email                   text
├── phone                   text
├── instagram_handle        text
├── is_iko_certified        boolean
├── is_verified             boolean default false
├── is_featured             boolean default false
├── offers_airport_transfer boolean default false
├── transfer_notes          text
├── discount_config         jsonb   -- {"duration": {"7": 10, "14": 15}}
├── status                  enum    pending | active | inactive
├── sanity_document_id      text
├── created_at              timestamptz
└── updated_at              timestamptz
```

### `prospects`
```sql
prospects
├── id                uuid    PK
├── handle            text
├── platform          enum    instagram | youtube | tiktok | web
├── segment           enum    operator | influencer | brand | photographer |
│                             coach | tourism_board | event_organizer | publication
├── follower_count    int
├── engagement_rate   float
├── location          text
├── bio               text
├── website           text
├── contact_email     text
├── relevance_score   float
├── tier              enum    tier_1 | tier_2 | tier_3
├── outreach_status   enum    not_contacted | draft_ready | contacted |
│                             responded | converted | not_interested
├── pitch_draft       text
├── notes             text
├── source            text
├── created_at        timestamptz
└── updated_at        timestamptz
```

### `spot_check_ins`
```sql
spot_check_ins
├── id          uuid    PK
├── user_id     uuid    FK → users.id
├── spot_id     uuid    FK → spots.id
├── visit_date  date
├── source      enum    trip_completion | manual
├── trip_id     uuid    FK → trips.id    nullable
└── created_at  timestamptz
```

---

## Logistics Engine Formula

```
total_kit_cost = (base_fee × bags_over_free_allowance)
               + sports_surcharge
               + oversize_fee
               + overweight_fee
```

Each term comes from a different column across `airline_bag_fees` and `airline_sports_fees`.
`allowance_concept` (piece vs weight) determines whether kite gear consumes the free allowance or adds to it.

---

## Env Vars

```bash
SUPABASE_URL=https://gynrvwjmddsycfzivijf.supabase.co
SUPABASE_SERVICE_KEY=<service role key — in tools/.env>
```

Client pattern in scripts:
```python
from supabase import create_client
client = create_client(os.environ["SUPABASE_URL"], os.environ["SUPABASE_SERVICE_KEY"])
```
