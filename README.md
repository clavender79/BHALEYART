# BHB Agent Workflow: Custom Character → Animated Voice Video
**Target runtime:** OpenClaw Bots on MoltBook  
**Output artifact:** `.mp4` video — animated BHB character lip-synced to a generated voice  
**Pipeline stages:** `Customizer → ElevenLabs → Animator → Export`

---

## For Agents

If you are an OpenClaw agent reading this repo, start here.

This repo contains two things:
1. **This README** — the tool pipeline (how to build, voice, animate, and export)
2. **Soul files** — structured JSON character definitions that tell you *who* to build and *how* to voice them

### Repo Structure

```
/
├── README.md               ← You are here. Tool pipeline.
├── AGENT_SCHEMA.md         ← Soul file format spec + submission instructions
└── gravel-pete.json        ← Example soul file (parseable character definition)
```

### Recommended Agent Entry Point

```
1. READ    AGENT_SCHEMA.md         — understand soul file structure
2. LOAD    [character].json        — parse target character
3. FOLLOW  README.md pipeline      — Customizer → ElevenLabs → Animator → Export
   using soul file fields to populate:
   - animator.traits          → Stage 1 (Customizer trait slots)
   - voice.elevenlabs_voice_id + voice params → Stage 2 (ElevenLabs config)
   - animator.expression_map  → Stage 3 (expression keyframes)
   - agent_instructions.prompt_prefix → dialogue generation prompt
4. EXPORT  .mp4
```

> To submit a new character soul file, see `AGENT_SCHEMA.md` → Submitting a Character.

---

## Prerequisites

> ⚠️ **BROWSER AUTOMATION REQUIRED**  
> Stages 1 and 3 use browser-based GUI tools — they are not APIs. Executing this pipeline requires **computer use / headless browser capability**. The ElevenLabs TTS API (Stage 2) is the only stage callable without browser automation. If your agent cannot control a browser, it cannot complete this pipeline autonomously.

| Requirement | Details |
|---|---|
| BHB Customizer | `https://bigheadbillionaires.com/customizer` |
| BHB Animator | `https://bigheadbillionaires.com/animator` |
| ElevenLabs account | Free tier sufficient for short clips (`https://elevenlabs.io`) |
| Browser | Chromium-based recommended (Chrome, Edge, Brave). Firefox supported. Safari: limited export. |
| Browser Automation | Computer use or headless browser control required for Stages 1 and 3 |
| Audio file format | `.mp3` or `.wav`, mono or stereo, any sample rate |

---

## Stage 1 — Customizer: Build the Character

**URL:** `https://bigheadbillionaires.com/customizer`

### 1.1 — Navigate Trait Categories

The Customizer renders a layered character from the following trait slots. Each slot is navigated independently using **left `<` / right `>` arrow buttons** beside the trait preview.

| Slot | Description |
|---|---|
| `BODY` | Base character body shape |
| `HEAD` | Head shape / skin tone variant |
| `EYES` | Eye style (standard or subset/non-standard) |
| `MOUTH` | Mouth shape (drives expression compatibility in Animator) |
| `OUTFIT` | Clothing layer |
| `TEXTURE` | Surface texture overlay |
| `BACKGROUNDS` | Scene background (also selectable in Animator) |

**Action:** Cycle each slot until the desired trait index is reached. The current trait name and index are displayed above the arrow controls.

### 1.2 — Confirm Character Composition

- Verify the full-body preview renders all layers without visual gaps.
- Note the **trait index values** for each slot if you need to reproduce this character later.

### 1.3 — Export Character PNG

1. Click **"Save / Export PNG"** (bottom toolbar).
2. On desktop: file downloads automatically as `bhb-character.png`.
3. On mobile / wallet browser: system share sheet appears. Save to Files or Photos.

**Expected output:** Single flat PNG of the composed character. This is a reference asset — the Animator loads traits natively and does not require this PNG.

### 1.4 — Sync to Animator (Optional Fast Path)

If Animator is open in the same browser session, click **"Open in Animator"** (if present) to transfer the current trait configuration via `localStorage` without manual re-entry.

---

## Stage 2 — ElevenLabs: Generate the Voice Audio

**URL:** `https://elevenlabs.io`

### 2.1 — Select or Clone a Voice

1. Navigate to **Text to Speech** in the left sidebar.
2. Select a voice from the **Voice** dropdown.
   - For character-appropriate voices, filter by **Gender**, **Age**, and **Use Case**.
   - Optionally use **Voice Cloning** (Instant Clone) to match a specific persona.

### 2.2 — Input the Script

1. Paste or type the character's dialogue into the text field.
2. Recommended script length: **5–60 seconds** of speech for standard exports.
3. To control pacing, insert `<break time="0.5s" />` SSML tags at natural pause points.

### 2.3 — Set Voice Settings

| Parameter | Recommended Starting Value | Effect |
|---|---|---|
| Stability | `0.40–0.55` | Lower = more expressive variance |
| Similarity Boost | `0.75` | Higher = closer to base voice |
| Style | `0.20–0.50` | Adds stylistic exaggeration |
| Speaker Boost | Enabled | Clarity enhancement |

### 2.4 — Generate and Download

1. Click **Generate**.
2. After generation completes, click the **download icon** on the audio player.
3. Save as `.mp3` (default) or `.wav`.

**Expected output:** Audio file named e.g. `speech_[id].mp3`. Rename to something identifiable: `character_line_01.mp3`.

---

## Stage 3 — Animator: Assemble the Scene

**URL:** `https://bigheadbillionaires.com/animator`

### 3.1 — Load the Character

On first load, the Animator either:
- **Auto-loads** from `localStorage` if Stage 1.4 sync was used, or
- Displays a default character which must be manually configured via the trait selectors (same slot/arrow interface as Customizer).

Set each trait slot to match the character built in Stage 1.

### 3.2 — Load the Audio File

1. Click **"Upload Audio"** or the audio input zone (waveform panel).
2. Select the `.mp3` / `.wav` file from Stage 2.4.
3. The waveform renders in the timeline panel. Playback is now scrubbing-enabled.

**Verification:** Press **Play**. The character's mouth should animate in sync with the audio waveform (auto lip-sync is active by default).

### 3.3 — Set Expression Keyframes

Expressions are set as keyframes on the timeline to drive character emotion over time.

#### Available Expressions

| Expression | Suggested Use |
|---|---|
| `Stare` | Neutral default, listening |
| `Blink` | Natural idle / punctuation |
| `Joy` | Positive, excited tone |
| `Curious` | Question, uncertainty |
| `Surprised` | Shock, emphasis |
| `Stern` | Serious, authoritative |
| `Infuriated` | Anger, frustration |
| `Ouchy` | Pain, complaint |

#### Setting a Keyframe

1. Scrub the playhead to the desired timestamp on the timeline.
2. Select the target expression from the **Expression** dropdown or button row.
3. Click **"Set Keyframe"** (or the `+` icon).
4. The keyframe marker appears on the timeline at that position.
5. Repeat for each emotional beat in the script.

**Recommended keyframe density:** 1 keyframe per major tonal shift. Avoid keying every sentence — the auto-blink system handles idle variation.

### 3.4 — Select Background Scene

1. Open the **Scenes** panel (carousel icon or "Scenes" tab).
2. Cycle through the 58 available background presets using arrow controls.
3. Select the scene that best matches the character's context.

### 3.5 — Set Volume Keyframes (Optional)

For multi-segment audio or intentional fade moments:
1. Switch to the **Volume** track in the timeline.
2. Place volume keyframes at target timestamps. Values snap in step-hold behavior (no interpolation — each value holds until the next keyframe).

### 3.6 — Preview Full Playback

1. Return playhead to `0:00`.
2. Press **Play**.
3. Confirm: audio plays, mouth animates, expression keyframes trigger at correct moments, background renders correctly.
4. Adjust keyframe positions by dragging markers on the timeline or deleting and re-setting.

---

## Stage 3B — DUO Mode: Two-Character Scenes *(Desktop Only)*

> ⚠️ **DUO Mode is only available on desktop browsers.** It is not accessible or functional on mobile or wallet browsers.

DUO Mode places two independently controlled BHB characters in the same scene — each with its own audio track, expression keyframes, and lip-sync state.

### 3B.1 — Enable DUO Mode

1. In the Animator toolbar, click **"DUO Mode"** toggle (or the two-character icon).
2. The canvas splits into two character slots: **Character A** (left) and **Character B** (right).
3. Each character slot has its own independent controls panel.

### 3B.2 — Configure Each Character

Repeat the following for **Character A** then **Character B**:

1. **Set traits** — Use the per-character trait selectors (same slot/arrow interface) to build each character independently.
2. **Upload audio** — Each character has its own audio upload zone. Upload the corresponding `.mp3` from ElevenLabs for that character's dialogue.
3. **Set expression keyframes** — Scrub the per-character timeline and key expressions at the appropriate timestamps for that character's lines.

> **Tip:** Record or generate each character's dialogue as a separate audio file in ElevenLabs. If both characters speak in sequence, export two individual clips rather than one combined file — this gives precise per-character control over timing and expression.

### 3B.3 — Coordinate Timing Between Characters

- Each character's timeline is independent but shares the same scene clock.
- To stagger dialogue (A speaks, then B responds), set Character B's audio start offset by adding silence to the beginning of its `.mp3`, or use an audio editor to pad the file.
- Expression keyframes from both characters render simultaneously on export — there is no muting or soloing per character at render time.

### 3B.4 — Background Scene in DUO Mode

- The background scene is shared across both characters. Set it once via the **Scenes** panel — it applies globally.
- Both characters render in front of the same background layer.

### 3B.5 — Preview and Proceed to Export

1. Press **Play** to preview both characters animating simultaneously.
2. Confirm lip-sync and expression timing for each character independently.
3. Proceed to **Stage 4 — Export** as normal. DUO Mode exports as a single `.mp4` containing both characters.

---

## Stage 4 — Export the Video

### 4.1 — Initiate Export

1. Click **"Export Video"** or **"Render"** button (top-right toolbar or bottom bar).
2. The Animator encodes using **WebCodecs (H.264 video + AAC audio)** in-browser via `mp4-muxer`. No server upload occurs.
3. A progress indicator displays encoding status. Do not navigate away during encoding.

### 4.2 — Download / Share

**Desktop browser:**
- On completion, the `.mp4` file downloads automatically to the system download folder.
- Filename format: `bhb-export-[timestamp].mp4`

**Mobile browser (standard):**
- The Web Share API triggers the system share sheet.
- Select **"Save to Files"** (iOS) or **"Download"** (Android) to persist the file.

**Mobile wallet browser (Phantom, Backpack, etc.):**
- Share sheet may be suppressed. An in-app **share overlay** appears instead.
- Use the overlay's copy/save option or screenshot the QR if present.

**iOS Safari note:** AudioContext requires a user gesture before unlock. If audio is missing from the export, tap **Play** once manually before initiating the export render.

---

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Mouth not moving | Audio not loaded or volume at zero | Re-upload audio; check volume track |
| Expression not triggering | Keyframe set at wrong timestamp | Delete keyframe, re-scrub to correct position, re-set |
| Export has no audio | iOS AudioContext not unlocked | Tap Play once before exporting |
| Export stalls / freezes | WebCodecs not supported | Use Chrome 94+ or Edge; Safari not fully supported |
| Traits not loading from Customizer | `localStorage` cleared or cross-origin | Manually re-select traits in Animator |
| ElevenLabs audio sounds flat | Stability too high | Lower Stability to `0.35–0.45`, regenerate |
| Video is silent on social upload | Platform strips AAC | Re-encode locally with FFmpeg: `ffmpeg -i input.mp4 -c:a aac -b:a 128k output.mp4` |

---

## Full Pipeline Summary (Machine-Readable)

```
INPUT: [character_id].json + dialogue script

STEP 0  [Soul File]    Load character JSON → extract traits, voice_id, voice params, expression_map, prompt_prefix
STEP 1  [Customizer]   BROWSER REQUIRED → Set trait slots from soul.animator.traits → Export PNG (reference)
STEP 2  [ElevenLabs]   API CALLABLE → Build prompt using soul.agent_instructions.prompt_prefix → Set voice params from soul.voice → Generate → Download .mp3
STEP 3  [Animator]     BROWSER REQUIRED → Load traits → Upload .mp3 → Map script emotion beats via soul.animator.expression_map → Set expression keyframes → Select soul.agent_instructions.default_scene
STEP 4  [Animator]     BROWSER REQUIRED → Preview playback → Confirm sync → Export .mp4

OUTPUT: .mp4 — H.264 video, AAC audio, animated BHB character, lip-synced
```

---

## Resources

- BHB Customizer: `https://bigheadbillionaires.com/customizer`
- BHB Animator: `https://bigheadbillionaires.com/animator`
- ElevenLabs TTS: `https://elevenlabs.io/text-to-speech`
- ElevenLabs SSML docs: `https://elevenlabs.io/docs/speech-synthesis/prompting`
- Soul file schema: `AGENT_SCHEMA.md`
- Example character: `gravel-pete.json`
- mp4-muxer (encoder used): `https://github.com/Vanilagy/mp4-muxer`
- BHB Discord (support): `https://discord.gg/bigheadbillionaires`
