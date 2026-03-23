---
name: fairstack-dubbing
description: |
  Translate video or audio to another language via the FairStack API.
  AI dubbing powered by ElevenLabs via fal.ai. Supports video and audio input.
  Trigger on: dub, dubbing, translate video, translate audio, localize,
  multilingual, voice translation, language translation audio.
---

# AI Dubbing

Translate video or audio speech into another language with AI dubbing. Preserves speaker voice characteristics while changing the language.

**Base URL:** `https://fairstack.ai`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Generate Dubbing

### `POST /v1/dub/generate`

Dub a video or audio file into a target language.

```bash
curl -X POST https://fairstack.ai/v1/dub/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "video_url": "https://example.com/my-video.mp4",
    "target_lang": "es"
  }'
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `video_url` | string | Yes* | URL of the video to dub |
| `audio_url` | string | Yes* | URL of the audio to dub (alternative to video) |
| `target_lang` | string | **Yes** | Target language code (`es`, `fr`, `ja`, `de`, `pt`, `zh`, etc.) |
| `source_lang` | string | No | Source language code (auto-detected if omitted) |
| `num_speakers` | number | No | Number of speakers (1-10, auto-detected if omitted) |

*Either `video_url` or `audio_url` is required.

### Response (async — 202 for queued jobs)

```json
{
  "data": {
    "generation_id": "gen_abc123",
    "status": "queued",
    "message": "Dubbing job queued. Poll /v1/jobs/:id for status."
  }
}
```

### Response (completed — 201)

```json
{
  "data": {
    "generation_id": "gen_abc123",
    "status": "completed",
    "output_url": "https://media.fairstack.ai/dubs/gen_abc123.mp4",
    "model": "fal-ai/elevenlabs/dubbing",
    "cost": 0.15,
    "cost_micro": 150000,
    "currency": "USD"
  }
}
```

---

## JavaScript / TypeScript

```typescript
import FairStack from "@fairstack/sdk";

const fs = new FairStack({ apiKey: process.env.FAIRSTACK_API_KEY! });

// Dub a video to Spanish
const gen = await fs.dubbing.generate({
  source_url: "https://example.com/my-video.mp4",
  target_language: "es",
});

// If queued, poll for completion
if (gen.status !== "succeeded") {
  const result = await fs.generations.get(gen.id);
  console.log(result.output?.url);
}
```

## Python

```python
import requests, os, time

headers = {
    "Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}",
    "Content-Type": "application/json",
}

resp = requests.post(
    "https://fairstack.ai/v1/dub/generate",
    headers=headers,
    json={
        "video_url": "https://example.com/my-video.mp4",
        "target_lang": "es",
    },
)
data = resp.json()["data"]

# Poll if queued
if data["status"] == "queued":
    while True:
        time.sleep(10)
        poll = requests.get(
            f"https://fairstack.ai/v1/generations/{data['generation_id']}",
            headers=headers,
        )
        result = poll.json()
        if result["status"] in ("succeeded", "failed"):
            break

    print(result.get("output", {}).get("url"))
```

---

## Supported Languages

Common language codes: `en` (English), `es` (Spanish), `fr` (French), `de` (German), `it` (Italian), `pt` (Portuguese), `ja` (Japanese), `ko` (Korean), `zh` (Chinese), `ar` (Arabic), `hi` (Hindi), `ru` (Russian), `tr` (Turkish), `nl` (Dutch), `pl` (Polish), `sv` (Swedish).

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits |
| 422 | `validation_error` | Missing video_url/audio_url or invalid target_lang |
| 429 | — | Rate limited |
| 503 | `provider_error` | Dubbing model temporarily unavailable |

---

## Pricing

- **Dubbing cost:** Based on audio duration (~$0.002/second)
- Cost is estimated upfront; actual cost depends on source duration
- Use `POST /v1/estimate` with `modality: "dub"` for pricing
