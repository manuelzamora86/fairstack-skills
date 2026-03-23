---
name: fairstack-model-compare
description: |
  Run side-by-side model comparisons via the FairStack API.
  Same prompt across 2-4 models for image, voice, video, or music.
  Estimate-only mode available. Trigger on: compare models, side by side,
  which is better, A/B test models, model comparison, model shootout, versus.
---

# Model Compare

Run the same prompt through 2-4 models side-by-side. Compare quality, speed, and cost across image, voice, video, or music models. Supports estimate-only mode to check costs before generating.

**Base URL:** `https://fairstack.ai`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Image Compare

### `POST /v1/image/compare`

Compare 2-4 image models with the same prompt.

```bash
curl -X POST https://fairstack.ai/v1/image/compare \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A photorealistic portrait of a woman in golden hour light",
    "models": ["gpt-image-1.5-t2i", "seedream-v4-t2i", "imagen-4-t2i"],
    "confirm": false
  }'
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | **Yes** | Text prompt for all models |
| `models` | string[] | **Yes** | 2-4 model slugs to compare |
| `confirm` | boolean | No | `false` = estimate only, `true` or omit = generate |
| `width` | number | No | Image width (models that support it) |
| `height` | number | No | Image height |
| `aspect_ratio` | string | No | Aspect ratio |

### Estimate Response (`confirm: false`)

```json
{
  "data": {
    "per_model": [
      { "model": "gpt-image-1.5-t2i", "cost": { "infra": 0.008, "margin": 0.002, "total": 0.009 } },
      { "model": "seedream-v4-t2i", "cost": { "infra": 0.004, "margin": 0.001, "total": 0.005 } },
      { "model": "imagen-4-t2i", "cost": { "infra": 0.040, "margin": 0.008, "total": 0.048 } }
    ],
    "total": 0.062,
    "total_micro": 62000
  }
}
```

---

## Voice Compare

### `POST /v1/voice/compare`

Compare 2-4 TTS models with the same text.

```bash
curl -X POST https://fairstack.ai/v1/voice/compare \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "The quick brown fox jumps over the lazy dog.",
    "models": ["kokoro-tts", "qwen-3-tts", "minimax-speech-2-8-hd"],
    "voice": "alloy"
  }'
```

### Additional Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | string | **Yes** | Text to synthesize |
| `models` | string[] | **Yes** | 2-4 voice model slugs |
| `voice` | string | No | Voice ID (uses each model's default if omitted) |
| `format` | string | No | `mp3` or `wav` |
| `speed` | number | No | Speech speed multiplier |
| `confirm` | boolean | No | `false` = estimate only |

---

## Video Compare

### `POST /v1/video/compare`

Compare 2-4 video models with the same prompt.

```bash
curl -X POST https://fairstack.ai/v1/video/compare \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A drone shot flying over a mountain lake at sunrise",
    "models": ["sora-2-10s", "kling-v2-5s", "veo-3-1-fast"],
    "confirm": false
  }'
```

### Additional Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | **Yes** | Text prompt |
| `models` | string[] | **Yes** | 2-4 video model slugs |
| `duration_sec` | number | No | Video duration (default: 5) |
| `aspect_ratio` | string | No | `16:9`, `9:16`, `1:1` |
| `confirm` | boolean | No | `false` = estimate only |

---

## Music Compare

### `POST /v1/music/compare`

Compare 2-4 music models with the same prompt.

```bash
curl -X POST https://fairstack.ai/v1/music/compare \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Upbeat indie pop with acoustic guitar and whistling",
    "models": ["suno-v4", "mureka-v7.6"],
    "duration_sec": 30,
    "confirm": false
  }'
```

### Additional Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | **Yes** | Music description |
| `models` | string[] | **Yes** | 2-4 music model slugs |
| `duration_sec` | number | No | Duration (default: 90) |
| `lyrics` | string | No | Lyrics for vocal music |
| `confirm` | boolean | No | `false` = estimate only |

---

## JavaScript / TypeScript

```typescript
import FairStack from "@fairstack/sdk";

const fs = new FairStack({ apiKey: process.env.FAIRSTACK_API_KEY! });

// Image compare
const images = await fs.compare.image({
  prompt: "A sunset over mountains",
  models: ["gpt-image-1.5-t2i", "seedream-v4-t2i"],
});
for (const gen of images) {
  console.log(`${gen.model}: ${gen.output?.url}`);
}

// Voice compare
const voices = await fs.compare.voice({
  text: "Hello world",
  models: ["kokoro-tts", "qwen-3-tts"],
});

// Video compare (estimate only)
const videoEst = await fs.compare.video({
  prompt: "Ocean waves at sunset",
  models: ["sora-2-10s", "kling-v2-5s"],
  confirm: false,
});
console.log(`Total estimated: $${videoEst.total}`);

// Music compare
const songs = await fs.compare.music({
  prompt: "Chill lo-fi hip hop beat",
  models: ["suno-v4", "mureka-v7.6"],
  duration_sec: 30,
});
```

## Python

```python
import requests, os

headers = {
    "Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}",
    "Content-Type": "application/json",
}

# Image compare (estimate only)
resp = requests.post(
    "https://fairstack.ai/v1/image/compare",
    headers=headers,
    json={
        "prompt": "A photorealistic landscape",
        "models": ["gpt-image-1.5-t2i", "seedream-v4-t2i", "imagen-4-t2i"],
        "confirm": False,
    },
)
data = resp.json()["data"]
for m in data["per_model"]:
    print(f"  {m['model']}: ${m['cost']['total']}")
print(f"Total: ${data['total']}")
```

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits for all models |
| 422 | `validation_error` | Unknown model slug, too many/few models (2-4) |
| 429 | — | Rate limited |
| 503 | `SERVICE_UNAVAILABLE` | One or more providers down |

---

## Pricing

- **Estimate mode:** Free (`confirm: false`)
- **Generate mode:** Sum of individual model costs
- Each model charges its standard per-generation rate
- Use estimate mode first to compare costs before committing
