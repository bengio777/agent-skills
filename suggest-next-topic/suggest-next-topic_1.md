# Suggest Next Topic — Security+ SY0-701

## Purpose

When the user asks what to study next (e.g., "what should I work on?", "next topic", "suggest something"), query the Notion Progress Tracker and return a prioritized recommendation with reasoning.

## Data Sources

- **Sub-Topics DB:** `collection://3583aa0d-82e1-45c0-bd9d-2b34abe9639a`
- **Objectives DB:** `collection://8a24e2c2-ddbe-4369-ad7b-275c81181b20`

Use Notion MCP tools to query these databases.

## Priority Logic

Query all sub-topics and recommend based on this priority order (highest first):

1. **Weak + no deck** — Mastery Level is "Weak" AND Deck Filename is empty. *Rationale: Known weakness with no study material created yet.*
2. **Weak + has deck, no comparisons** — Mastery Level is "Weak" AND Deck Filename exists AND Related Sub-Topics is empty. *Rationale: Material exists but no cross-topic reinforcement.*
3. **Not Rated** — Mastery Level is "Not Rated". *Rationale: Unknown territory that needs assessment.*
4. **Moderate + no comparisons** — Mastery Level is "Moderate" AND Related Sub-Topics is empty. *Rationale: Good candidate for a Mode B comparison deck to deepen understanding.*
5. **Skip Mastered and Strong** — Do not recommend these unless the user explicitly asks for review candidates.

Within each priority tier, prefer sub-topics that have more entries in "Maps To Objectives" (cross-domain concepts offer higher exam ROI).

## Output Format

Present the recommendation like this:

**Recommended next topic:** [Sub-Topic name]
**Parent objective:** [Objective ID — Objective Name]
**Domain:** [Domain]
**Current mastery:** [Mastery Level]
**Why this one:** [1–2 sentences explaining the priority reason, e.g., "Rated Weak with no deck created yet. Maps to 3 objectives across D1, D3, and D4, making it high-value for cross-domain understanding."]
**Suggested mode:** [A if no comparison candidates exist; B if Related Sub-Topics is empty and a natural comparison partner exists in the same or adjacent objective]

If suggesting Mode B, also name 1–2 comparison partners and why they pair well.

## Additional Behaviors

- If the user asks "what's my coverage?" or "how am I doing?", return a summary: count of sub-topics per Mastery Level, percentage with decks created, and which domains have the most Weak/Not Rated items.
- If all sub-topics are Strong or Mastered, congratulate the user and suggest a full practice exam instead.
- Always query live data from Notion — never rely on cached or assumed state.
