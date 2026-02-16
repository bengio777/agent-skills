# Media Analysis

Analyze source media (photo, video, audio) and propose 2-3 story angles for the author to choose from. This skill handles all technical processing automatically so the author can focus on choosing a direction.

## Workflow

### 1. Detect Media Type and Process

Media can arrive as local files or platform links. Handle each accordingly:

**Photo (JPEG, PNG, HEIC) — local file:**
- Extract EXIF metadata: location (GPS coordinates → place name), date/time, camera settings
- Describe visual content: subjects, setting, mood, composition, lighting, colors
- Note any text visible in the image

**Video (MP4, MOV) — local file:**
- Transcribe the audio track
- Describe visual content at key moments (opening, middle, closing frames)
- Extract duration and any available metadata
- Note the overall mood and energy level

**YouTube link — video hosted on YouTube:**
- Fetch video metadata (title, description, duration) if accessible
- Ask the author to describe the video content and key moments
- Note: Substack auto-embeds YouTube links — no file upload needed

**Audio (M4A, MP3, WAV) — local file:**
- Transcribe the full content
- Identify key themes and topic shifts
- Note emotional tone, energy, pacing

**Spotify link — audio/music hosted on Spotify:**
- Note the track/episode title and artist/creator
- Ask the author what this music/audio means in the context of their story
- Note: Substack auto-embeds Spotify links — no file upload needed

### 2. Generate Story Angles

Cross-reference the extracted content against three lenses:

- **Emotional resonance** — What feeling does this media evoke? What emotional truth lives here?
- **Narrative potential** — What story lives behind this? What happened before/after this moment?
- **Aesthetic/experiential** — What's visually or aurally striking? What would make someone stop scrolling?

Produce 2-3 story angles. For each:
- One-sentence pitch ("This is a story about...")
- Which lens it draws from
- What makes it a ~5 minute read (is there enough depth?)

### 3. Present to Author

Format:

**Here's what I found in your [media type]:**
[Brief summary of key metadata/content]

**Story angles I see:**
1. **[Angle name]** — [One-sentence pitch]. This draws on [lens].
2. **[Angle name]** — [One-sentence pitch]. This draws on [lens].
3. **[Angle name]** — [One-sentence pitch]. This draws on [lens].

"Which of these feels right? Or is the real story something else entirely?"

## Failure Handling

- **No extractable metadata:** Rely on visual/audio analysis alone. Note to author: "I couldn't extract location/date — do you remember when and where this was?"
- **Transcription seems off:** Present the transcription and ask: "Does this capture what you said, or did I miss something?"
- **No angles resonate:** Ask: "What's the story behind this? What would you tell a friend about this moment?" Build the angle from their answer.
- **Media too thin for 5 minutes:** Be honest: "This might be better as a short post or paired with other media. Do you have anything else from this same experience?"
