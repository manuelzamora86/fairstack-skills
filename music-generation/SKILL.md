---
name: fairstack-music-generation
description: |
  Generate music via the FairStack API. AI music composition with lyrics,
  tempo/BPM, key signature, and time signature controls. Song covers, remixes,
  and editing (repaint, retake, extend). Models include Mureka and ACE-Step.
  Async by default. Trigger on: generate music, create music, AI music, compose
  song, soundtrack, background music, jingle, song cover, music generation.
---

# Music Generation

Compose original music with AI — full songs with lyrics, instrumentals, covers, and remixes. Control tempo, key, and style through a single endpoint.

**Base URL:** `https://fairstack.ai`
**Endpoint:** `POST /v1/music/generate`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Quick Start

Music generation is **async by default** — submit and poll.

### curl

```bash
# 1. Submit music generation
curl -X POST https://fairstack.ai/v1/music/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Upbeat indie pop with acoustic guitar and handclaps",
    "duration_sec": 60,
    "bpm": 120,
    "key": "G major",
    "project": "podcast",
    "tags": [
      {"key": "tag", "value": "intro"},
      {"key": "tag", "value": "upbeat"},
      {"key": "tag", "value": "indie-pop"}
    ]
  }'

# Response: {"data": {"generation_id": "gen_music_123", "status": "running", ...}}

# 2. Poll for completion
curl https://fairstack.ai/v1/generations/gen_music_123 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### CLI

```bash
fairstack generate music \
  --prompt "Upbeat indie pop with acoustic guitar and handclaps" \
  --duration 60 --bpm 120 --key "G major" \
  --project podcast \
  --tags "intro, upbeat, indie-pop"
```

### JavaScript / TypeScript

```typescript
// Submit
const submit = await fetch("https://fairstack.ai/v1/music/generate", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FAIRSTACK_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    prompt: "Upbeat indie pop with acoustic guitar and handclaps",
    lyrics: "[Verse 1]\nWalking down the street on a sunny day\nNothing gonna stand in my way\n\n[Chorus]\nOh oh oh, feeling alive\nOh oh oh, ready to thrive",
    duration_sec: 90,
    bpm: 120,
    key: "G major",
  }),
});

const { data: job } = await submit.json();

// Poll until complete
async function pollUntilDone(id: string) {
  while (true) {
    const res = await fetch(`https://fairstack.ai/v1/generations/${id}`, {
      headers: { Authorization: `Bearer ${process.env.FAIRSTACK_API_KEY}` },
    });
    const { data } = await res.json();
    if (data.status === "succeeded") return data;
    if (data.status === "failed") throw new Error(data.error || "Failed");
    await new Promise((r) => setTimeout(r, 5000));
  }
}

const result = await pollUntilDone(job.generation_id);
console.log(result.url);      // CDN URL to audio
console.log(result.cost_usd); // e.g. 0.60
```

### Python

```python
import requests, os, time

headers = {"Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}"}

# Submit
resp = requests.post(
    "https://fairstack.ai/v1/music/generate",
    headers=headers,
    json={
        "prompt": "Upbeat indie pop with acoustic guitar and handclaps",
        "duration_sec": 60,
        "bpm": 120,
        "key": "G major",
    },
)
job = resp.json()["data"]

# Poll
while True:
    status = requests.get(
        f"https://fairstack.ai/v1/generations/{job['generation_id']}",
        headers=headers,
    ).json()["data"]
    if status["status"] == "succeeded":
        print(status["url"])
        break
    if status["status"] == "failed":
        raise Exception("Failed")
    time.sleep(5)
```

---

## Response

### Submit Response (202)

```json
{
  "request_id": "req_abc123",
  "data": {
    "generation_id": "gen_music_123",
    "status": "running",
    "poll_url": "/v1/generations/gen_music_123",
    "estimated_cost": 0.60,
    "estimated_cost_micro": 600000,
    "currency": "USD"
  }
}
```

### Completed Response

```json
{
  "request_id": "req_def456",
  "data": {
    "id": "gen_music_123",
    "status": "succeeded",
    "url": "https://media.fairstack.ai/generations/gen_music_123.mp3",
    "model": "mureka-v7.6",
    "duration_sec": 60,
    "cost_micro": 600000,
    "cost_usd": 0.60
  }
}
```

---

## Parameters

### `POST /v1/music/generate`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | **Yes** | Style description / genre tags (max 25,000 chars) |
| `lyrics` | string | No | Song lyrics with section markers `[Verse]`, `[Chorus]`, etc. (max 10,000 chars) |
| `model` | string | No | Model slug (default: `"mureka-v7.6"`) |
| `duration_sec` | number | No | Track length 10–600 seconds (default 90) |
| `bpm` | number | No | Tempo 40–240 BPM |
| `key` | string | No | Musical key (e.g. `"C major"`, `"A minor"`, `"F# minor"`) |
| `time_signature` | string | No | `"4/4"`, `"3/4"`, `"6/8"`, `"2/4"` |
| `language` | string | No | ISO 639-1 language code (e.g. `"en"`, `"es"`, `"ja"`) |
| `seed` | number | No | Reproducibility seed |
| `inference_steps` | number | No | Quality steps 1–100 |
| `guidance_scale` | number | No | Prompt adherence 1–30 |
| `project` | string | No | Project slug or ID to associate with this generation |
| `tags` | array | No | Key-value tags for organization (max 20). Format: `[{"key":"tag","value":"intro"}]` |
| `confirm` | boolean | No | Set `false` for cost quote only |
| `quote_id` | string | No | Confirm a previously created quote |

### Sync Mode

Pass `wait=true` to block until music generation completes:

```bash
curl -X POST "https://fairstack.ai/v1/music/generate?wait=true" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Lo-fi hip hop study beats", "duration_sec": 30}'
```

---

## Models

| Model | Cost | Duration | Best for |
|-------|------|----------|----------|
| `mureka-v7.6` | ~$0.50/90s | 10–600s | Full-featured: lyrics, style control, highest quality |
| `ace-step` | ~$0.15/30s | 10–300s | Instrumental, fine-grained BPM/key control |
| `suno-v4` | ~$0.05/30s | 10–240s | Quick generation, good for iteration |

Use `GET /v1/models?modality=music` for the full catalog with current pricing.

---

## Additional Endpoints

### Song Cover

Create a cover version of a song in a new style:

```bash
curl -X POST https://fairstack.ai/v1/music/cover \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Acoustic folk version with fingerpicked guitar",
    "reference_audio_url": "https://example.com/original-song.mp3",
    "reference_strength": 0.6,
    "duration_sec": 120
  }'
```

### Music Editing

Edit existing music — repaint sections, retake, or extend:

```bash
# Repaint a section (replace 30-60s with new content)
curl -X POST https://fairstack.ai/v1/music/edit \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "repaint",
    "source_audio_url": "https://example.com/my-track.mp3",
    "prompt": "Add a guitar solo here",
    "repaint_start_sec": 30
  }'

# Extend a track
curl -X POST https://fairstack.ai/v1/music/edit \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "extend",
    "source_audio_url": "https://example.com/my-track.mp3",
    "prompt": "Continue with a bridge section"
  }'
```

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits. |
| 402 | `QUOTA_EXCEEDED` | Monthly spending cap reached. |
| 422 | `validation_error` | Invalid parameters (e.g. duration out of range). |
| 429 | — | Rate limited. 600 req/min. |
| 503 | `SERVICE_UNAVAILABLE` | Music generation not enabled or provider down. |

---

## Best Practices

### Always use project + tags

Every generation should include `project` and at least 3 tags. Tags help you recover, search, and filter your work later.

```bash
--project podcast --tags "intro, upbeat, indie-pop"
```

### Style consistency

This is the #1 quality issue users hit. Inconsistent musical styles across tracks sound jarring in a project.

1. **Define your sonic identity per project.** Lock in a genre, BPM range, and key signature. Use the same prompt structure for all tracks in a project.
2. **Use the same model for tonal consistency.** Different models have different timbres and production styles.
3. **Tag which style was used:** `--tags "lo-fi, study-beats, podcast"` so you can filter by style later.
4. **For agents:** Create a music brief file with genre, BPM, key, and prompt templates that your agent references before generating.

---

## Pricing

All prices are **infrastructure cost + 20% platform fee**. No hidden charges.

- Music pricing scales with duration
- Credits deducted upfront, refunded on failure
- Use `confirm: false` to check cost before generating
