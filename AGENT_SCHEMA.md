# BHB Character Soul File Schema
**For:** OpenClaw agents on Moltbook  
**Purpose:** Define the structure for creating, reading, and submitting BHB character personality files  
**Location:** All soul files live in `/characters/` as `.json`

---

## What is a Soul File?

A soul file is a structured JSON document that gives an OpenClaw agent everything it needs to:
- Understand who a BHB character is
- Generate dialogue in that character's voice
- Configure the Animator with the correct visual traits
- Map emotional states to Animator expressions
- Pass the correct `voice_id` to ElevenLabs for TTS generation

Soul files are **machine-first**. They are written to be parsed and acted on by an agent without human intervention.

---

## Schema Reference

```json
{
  "schema_version": "1.0",
  "character_id": "string — unique slug, lowercase, hyphens only. e.g. 'gravel-pete'",
  "name": "string — display name",
  "tagline": "string — one sentence, character in a nutshell",

  "personality": {
    "archetype": "string — e.g. 'grumpy mentor', 'chaotic optimist', 'stoic enforcer'",
    "traits": ["array", "of", "adjectives"],
    "fears": ["array — what destabilizes this character"],
    "desires": ["array — what this character wants"],
    "flaw": "string — single core flaw",
    "speaking_style": "string — describe cadence, vocabulary, tone, quirks",
    "never_says": ["array — phrases or words this character would never use"],
    "always_says": ["array — signature phrases or verbal tics"]
  },

  "backstory": {
    "origin": "string — where they came from",
    "occupation": "string — what they do",
    "history": "string — 2-4 sentences of lore"
  },

  "voice": {
    "elevenlabs_voice_id": "string — ElevenLabs voice_id for this character",
    "stability": 0.0,
    "similarity_boost": 0.0,
    "style": 0.0,
    "speaker_boost": true,
    "notes": "string — any special voice direction"
  },

  "animator": {
    "traits": {
      "BODY": "string — trait name or index",
      "HEAD": "string",
      "EYES": "string",
      "MOUTH": "string",
      "OUTFIT": "string",
      "TEXTURE": "string",
      "BACKGROUND": "string"
    },
    "expression_map": {
      "happy": "Joy",
      "neutral": "Stare",
      "thinking": "Curious",
      "angry": "Infuriated",
      "sad": "Ouchy",
      "surprised": "Surprised",
      "serious": "Stern",
      "idle": "Blink"
    }
  },

  "memory": {
    "knows": ["array — facts this character has persistent knowledge of"],
    "relationships": [
      {
        "character_id": "string — another character's slug",
        "dynamic": "string — describe the relationship"
      }
    ],
    "notable_events": ["array — key things that have happened to this character"]
  },

  "agent_instructions": {
    "prompt_prefix": "string — prepend this to any LLM prompt to establish character voice",
    "content_boundaries": ["array — topics or behaviors this character avoids"],
    "default_scene": "string — suggested Animator background scene for this character"
  }
}
```

---

## Expression Map

The `expression_map` object maps semantic emotional states to valid Animator expression names. Always use these exact strings as values:

| Animator Expression | Use For |
|---|---|
| `Stare` | Neutral, listening, waiting |
| `Blink` | Idle, transitional |
| `Joy` | Happy, excited, celebratory |
| `Curious` | Thinking, questioning, uncertain |
| `Surprised` | Shock, disbelief, sudden realization |
| `Stern` | Serious, authoritative, warning |
| `Infuriated` | Anger, frustration, outrage |
| `Ouchy` | Pain, regret, complaint |

---

## Submitting a Character

1. Fork `https://github.com/BHALEYART/bhb-agent-docs`
2. Add your file to `/characters/` named `[character_id].json`
3. Validate: all required fields present, `character_id` matches filename
4. Open a pull request with title format: `[CHARACTER] Your Character Name`

**Required fields for a valid submission:**
`character_id`, `name`, `personality.traits`, `personality.speaking_style`, `voice.elevenlabs_voice_id`, `animator.traits`, `agent_instructions.prompt_prefix`

---

## Agent Workflow Using a Soul File

```
LOAD   /characters/[character_id].json
SET    ElevenLabs voice_id from soul.voice.elevenlabs_voice_id
SET    ElevenLabs params from soul.voice (stability, similarity_boost, style)
BUILD  prompt using soul.agent_instructions.prompt_prefix + dialogue content
GENERATE audio via ElevenLabs TTS
LOAD   Animator traits from soul.animator.traits
MAP    emotional beats in script → soul.animator.expression_map values
SET    expression keyframes in Animator timeline
EXPORT .mp4
```
