---
name: fairstack-voice-generation
description: |
  Generate speech from text via the FairStack API. 168 built-in voices across
  12 archetypes, voice cloning from reference audio, emotion control, and
  speed adjustment. Models from $0.001/request. Supports MP3 and WAV output.
  Trigger on: generate voice, text to speech, TTS, voice clone, read this aloud,
  narration, voiceover, speech synthesis, audio generation.
---

# Voice Generation

Turn text into natural speech with 168 voices, voice cloning, and emotion control. From narrators to characters — all through a single endpoint.

**Base URL:** `https://fairstack.ai`
**Endpoint:** `POST /v1/voice/generate`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Quick Start

### curl

```bash
curl -X POST https://fairstack.ai/v1/voice/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Welcome to FairStack. AI generation at fair prices.",
    "voice": "internal_marco",
    "model": "chatterbox-turbo"
  }'
```

### JavaScript / TypeScript

```typescript
const response = await fetch("https://fairstack.ai/v1/voice/generate", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FAIRSTACK_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    text: "Welcome to FairStack. AI generation at fair prices.",
    voice: "internal_marco",
    model: "chatterbox-turbo",
  }),
});

const { data } = await response.json();
console.log(data.url);          // CDN URL to audio file
console.log(data.duration_sec); // e.g. 3.2
console.log(data.cost_usd);     // e.g. 0.0012
```

### Python

```python
import requests, os

resp = requests.post(
    "https://fairstack.ai/v1/voice/generate",
    headers={"Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}"},
    json={
        "text": "Welcome to FairStack. AI generation at fair prices.",
        "voice": "internal_marco",
        "model": "chatterbox-turbo",
    },
)

data = resp.json()["data"]
print(data["url"])          # CDN URL
print(data["duration_sec"]) # e.g. 3.2
```

---

## Response

```json
{
  "request_id": "req_abc123",
  "data": {
    "id": "gen_voice_456",
    "url": "https://media.fairstack.ai/generations/gen_voice_456.mp3",
    "model": "chatterbox-turbo",
    "duration_sec": 3.2,
    "cost_micro": 1200,
    "cost_usd": 0.0012,
    "created_at": "2026-03-23T10:15:00Z"
  }
}
```

---

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | string | **Yes** | Text to speak (1–5,000 chars) |
| `voice` | string | No | Voice ID from the library (default: `"default"`) |
| `model` | string | No | Model slug (default: `"qwen-3-tts"`) |
| `format` | string | No | Output format: `"mp3"` (default) or `"wav"` |
| `ref_audio_url` | string | No | Reference audio URL for voice cloning |
| `ref_text` | string | No | Transcript of reference audio (max 500 chars) |
| `emotion` | string | No | Emotion preset (e.g. `"happy"`, `"sad"`, `"angry"`) |
| `emotion_vector` | number[] | No | 8-dimensional emotion vector (0.0–1.0 each) for fine control |
| `instruct_text` | string | No | Describe voice style (e.g. `"A warm female voice, British accent"`) |
| `voice_description` | string | No | Describe the voice to synthesize (when not using voice ID) |
| `options.speed` | number | No | Speech rate 0.5–2.0 (default 1.0) |
| `options.stability` | number | No | Voice stability 0.0–1.0 |
| `options.similarity` | number | No | Voice similarity 0.0–1.0 |
| `options.style` | number | No | Style exaggeration 0.0–1.0 |
| `confirm` | boolean | No | Set `false` to get a cost quote without generating |
| `quote_id` | string | No | Confirm a previously created quote |

---

## Models

| Model | Cost | Speed | Best for |
|-------|------|-------|----------|
| `qwen-3-tts` | ~$0.001 | Fast | Default. Good quality, lowest cost |
| `chatterbox-turbo` | ~$0.002 | Fast | Natural conversational speech |
| `elevenlabs-turbo-2-5` | ~$0.003 | Fast | Premium quality, emotion support |
| `minimax-speech-2-8-hd` | ~$0.004 | Medium | High-definition, long-form narration |
| `cosyvoice-2-instruct` | ~$0.002 | Medium | Instruction-following voice synthesis |

Use `GET /v1/models?modality=voice` for the full catalog with current pricing.

---

## Voice Library

168 built-in voices across 12 archetypes. Browse and search:

### List All Voices

```bash
curl https://fairstack.ai/v1/voice \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Filter by Category

```bash
# By archetype
curl "https://fairstack.ai/v1/voice?category=narrator" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"

# By gender
curl "https://fairstack.ai/v1/voice?gender=female" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"

# Search by keyword
curl "https://fairstack.ai/v1/voice?search=british+warm" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Voice Response Shape

```json
{
  "voice_id": "internal_marco",
  "name": "Marco",
  "archetype": "narrator",
  "category": "narrator",
  "accent": "latin-american",
  "gender": "male",
  "tags": ["warm", "storytelling", "documentary"],
  "description": "Deep, warm narrator voice with Latin American accent",
  "preview_url": "https://media.fairstack.ai/voices/internal_marco.mp3"
}
```

### Archetypes

`narrator`, `hero`, `villain`, `mentor`, `sidekick`, `romantic`, `child`, `professional`, `mysterious`, `tough`, `comedic`, `announcer`

---

## Examples

### Voice Cloning

Clone any voice from a reference audio sample:

```bash
curl -X POST https://fairstack.ai/v1/voice/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "This is my cloned voice speaking new words.",
    "ref_audio_url": "https://example.com/my-voice-sample.mp3",
    "ref_text": "The original words spoken in the sample audio.",
    "model": "chatterbox-turbo"
  }'
```

### With Emotion

```bash
curl -X POST https://fairstack.ai/v1/voice/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "I just got the most incredible news!",
    "voice": "internal_sofia",
    "emotion": "happy",
    "model": "elevenlabs-turbo-2-5"
  }'
```

### Instruction-Based Voice

```bash
curl -X POST https://fairstack.ai/v1/voice/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Good evening. Tonight on the program, we explore the deep ocean.",
    "instruct_text": "A deep, authoritative male voice with a British BBC accent, speaking slowly and deliberately",
    "model": "cosyvoice-2-instruct"
  }'
```

### Async Mode

```bash
# Return 202 immediately, poll for result
curl -X POST "https://fairstack.ai/v1/voice/generate?wait=false" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"text": "A long narration...", "voice": "internal_marco"}'

# Poll
curl https://fairstack.ai/v1/generations/gen_voice_456 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits. Response includes `fund_url`. |
| 402 | `QUOTA_EXCEEDED` | Monthly spending cap reached. |
| 422 | `validation_error` | Invalid parameters (empty text, `ref_text` without `ref_audio_url`, etc.) |
| 429 | — | Rate limited. Voice generation: 600 req/min. |
| 503 | `SERVICE_UNAVAILABLE` | Provider temporarily unavailable. |

---

## Pricing

All prices are **infrastructure cost + 20% platform fee**. No hidden charges.

- Voice pricing is typically per-character or per-second depending on model
- Costs returned in every response as `cost_micro` and `cost_usd`
- Use `POST /v1/estimate` or `confirm: false` to check cost before generating
- `POST /v1/enhance-prompt` is **free** — enhance your text before generating
