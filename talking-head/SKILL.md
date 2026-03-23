---
name: fairstack-talking-head
description: |
  Generate lip-synced talking head videos from text + character image via the FairStack API.
  Chains TTS and lip sync into one request. Arcads competitor at $0.35 vs $11.
  Trigger on: talking head, avatar video, lip sync, spokesperson, UGC video,
  dialogue video, character video, lipsync.
---

# Talking Head / Dialogue Video

Generate lip-synced talking head videos from text and a character image. Chains text-to-speech and lip sync into a single API call. An Arcads alternative at $0.35 vs $11.

**Base URL:** `https://fairstack.ai`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Generate Talking Head Video

### `POST /v1/dialogue/generate`

Generate a lip-synced video of a character speaking your text.

```bash
curl -X POST https://fairstack.ai/v1/dialogue/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Welcome to our product demo! Let me show you how it works.",
    "voice": "internal_marco",
    "character_image_url": "https://example.com/avatar.jpg",
    "project": "product-demo",
    "tags": [
      {"key": "tag", "value": "spokesperson"},
      {"key": "tag", "value": "demo"},
      {"key": "tag", "value": "welcome"}
    ]
  }'
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | string | Yes* | Text for the character to speak (max 5,000 chars) |
| `audio_url` | string | No* | Pre-recorded audio URL (skip TTS, lip sync only) |
| `voice` | string | Yes* | Voice ID from voice library (required when `text` is provided) |
| `character_image_url` | string | Yes* | Character face image URL |
| `character_video_url` | string | No* | Character video URL (for video lip sync) |
| `tts_model` | string | No | TTS model slug (default: `kokoro-tts`) |
| `talking_head_model` | string | No | Lip sync model slug (default: `infinitetalk`) |
| `resolution` | string | No | `480p` or `720p` |
| `speakers` | array | No | Multi-speaker dialogue lines |
| `project` | string | No | Project slug or ID to associate with this generation |
| `tags` | array | No | Key-value tags for organization (max 20) |

*Either `text` + `voice` or `audio_url` is required. Either `character_image_url` or `character_video_url` is required.

### Response (async — 202)

```json
{
  "data": {
    "generation_id": "gen_abc123",
    "status": "queued",
    "estimated_cost": {
      "total_micro": 350000,
      "total_usd": 0.35
    }
  }
}
```

Poll `GET /v1/generations/:id` for completion.

---

## Estimate Cost

### `POST /v1/dialogue/estimate`

Get the cost estimate without generating.

```bash
curl -X POST https://fairstack.ai/v1/dialogue/estimate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "A 30-second product walkthrough script...",
    "voice": "internal_marco",
    "character_image_url": "https://example.com/avatar.jpg"
  }'
```

### Response

```json
{
  "estimated_credits_micro": 350000,
  "estimated_usd": 0.35,
  "model": "infinitetalk",
  "type": "dialogue"
}
```

---

## JavaScript / TypeScript

```typescript
import FairStack from "@fairstack/sdk";

const fs = new FairStack({ apiKey: process.env.FAIRSTACK_API_KEY! });

// Estimate cost first
const estimate = await fs.dialogue.estimate({
  text: "Welcome to our demo!",
  voice: "internal_marco",
  image_url: "https://example.com/avatar.jpg",
});
console.log(`Estimated cost: $${estimate.estimated_usd}`);

// Generate (waits up to 2 minutes)
const video = await fs.dialogue.generate({
  text: "Welcome to our product demo! Let me show you how it works.",
  voice: "internal_marco",
  image_url: "https://example.com/avatar.jpg",
});
console.log(video.output?.url);

// Or submit and poll manually
const handle = await fs.dialogue.submit({
  text: "Hello from the async path!",
  voice: "alloy",
  image_url: "https://example.com/avatar.jpg",
});
const result = await handle.wait(); // polls until complete
```

## Python

```python
import requests, os, time

headers = {
    "Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}",
    "Content-Type": "application/json",
}

# Generate
resp = requests.post(
    "https://fairstack.ai/v1/dialogue/generate",
    headers=headers,
    json={
        "text": "Welcome to our product demo!",
        "voice": "internal_marco",
        "character_image_url": "https://example.com/avatar.jpg",
    },
)
data = resp.json()["data"]

# Poll for completion
while data["status"] in ("queued", "running"):
    time.sleep(5)
    poll = requests.get(
        f"https://fairstack.ai/v1/generations/{data['generation_id']}",
        headers=headers,
    )
    data = poll.json()

print(data["output"]["url"])
```

---

## Use Cases

- **UGC-style ads** — Generate spokesperson videos from any character image
- **Product demos** — Script + avatar = instant walkthrough video
- **Multilingual content** — Same character, different voices/languages
- **Social media** — Quick talking head clips for TikTok, Reels, Shorts

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits |
| 422 | `validation_error` | Missing required fields or invalid input |
| 429 | — | Rate limited |
| 503 | `SERVICE_UNAVAILABLE` | Provider temporarily down |

---

## Best Practices

### Always use project + tags

Every talking head generation should include `project` and at least 3 tags. Tag the character and purpose so you can find all videos for a given spokesperson.

```bash
--project product-demo --tags "spokesperson, demo, welcome"
```

### Style consistency

This is the #1 quality issue users hit. Inconsistent character appearance or voice breaks viewer trust.

1. **Use the same character image and voice for all videos in a series.** Changing either mid-series creates a jarring experience.
2. **Define a character profile** (image URL, voice ID, TTS model) and reuse it across all generations in the project.
3. **Tag which character was used:** `{"key":"character","value":"marco-avatar"}` so you can filter by character later.
4. **For agents:** Create a character manifest file with image URL, voice ID, and model slug. Reference it before every generation.

---

## Pricing

- **Talking head video:** ~$0.35 per video (TTS + lip sync combined)
- **Compare:** Arcads charges $11 per video -- FairStack is 30x cheaper
- Use `POST /v1/dialogue/estimate` to get exact cost before generating
