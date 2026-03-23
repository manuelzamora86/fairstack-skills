---
name: fairstack-models
description: |
  Browse the FairStack model catalog via API. List all 270+ models across
  6 modalities (image, video, voice, music, talking head, 3D), filter by
  modality or type, get detailed model info with pricing, capabilities,
  and supported parameters. No authentication required for listing.
  Trigger on: list models, available models, what models, model info,
  model catalog, model details, model pricing, browse models, model list.
---

# Model Catalog

Browse 270+ AI models across 6 modalities. Filter, search, and get detailed pricing and capability info — all through the public API.

**Base URL:** `https://fairstack.ai`
**Endpoints:** `GET /v1/models`, `GET /v1/models/:slug`

---

## No Auth Required

The models endpoint is public. Authentication is optional (but recommended for rate limit benefits).

---

## Quick Start

### List All Models

```bash
curl https://fairstack.ai/v1/models
```

### Filter by Modality

```bash
# Image models only
curl "https://fairstack.ai/v1/models?modality=image"

# Video models only
curl "https://fairstack.ai/v1/models?modality=video"

# Voice models only
curl "https://fairstack.ai/v1/models?modality=voice"

# Music models only
curl "https://fairstack.ai/v1/models?modality=music"
```

### Get Model Details

```bash
curl https://fairstack.ai/v1/models/seedream-v4-t2i
```

### Batch Lookup

```bash
curl "https://fairstack.ai/v1/models?ids=seedream-v4-t2i,sora-2-10s,mureka-v7.6"
```

---

## JavaScript / TypeScript

```typescript
// List all image models
const response = await fetch("https://fairstack.ai/v1/models?modality=image");
const { data } = await response.json();

for (const model of data.image) {
  console.log(`${model.slug}: $${model.price_usd} — ${model.tagline}`);
}

// Get specific model detail
const detail = await fetch("https://fairstack.ai/v1/models/sora-2-10s");
const { data: modelInfo } = await detail.json();
console.log(modelInfo.description);
console.log(modelInfo.pricing);
console.log(modelInfo.parameters);
```

## Python

```python
import requests

# List video models
resp = requests.get("https://fairstack.ai/v1/models", params={"modality": "video"})
models = resp.json()["data"]["video"]

for m in models:
    print(f"{m['slug']}: ${m['price_usd']} — {m['tagline']}")

# Get model detail
detail = requests.get("https://fairstack.ai/v1/models/imagen-4-t2i")
info = detail.json()["data"]
print(info["description"])
```

---

## List Response

```json
{
  "request_id": "req_abc123",
  "data": {
    "image": [
      {
        "slug": "z-image-turbo",
        "name": "Z-Image Turbo",
        "type": "t2i",
        "provider": "kieai",
        "price_micro": 4800,
        "price_usd": 0.0048,
        "tagline": "Ultra-fast image generation at the lowest cost",
        "is_cheapest": true,
        "parameters": { "aspect_ratio": ["1:1", "4:3", "16:9", "9:16"], "max_steps": 8 }
      },
      {
        "slug": "gpt-image-1.5-t2i",
        "name": "GPT Image 1.5",
        "type": "t2i",
        "provider": "falai",
        "price_micro": 10800,
        "price_usd": 0.0108,
        "tagline": "Strong text rendering and instruction following"
      }
    ],
    "video": [...],
    "voice": [...],
    "music": [...],
    "quality_tiers": {
      "image": {
        "economy": { "model": "z-image-turbo", "price_usd": 0.0048 },
        "smart": { "model": "gpt-image-1.5-t2i", "price_usd": 0.0108 },
        "best": { "model": "imagen-4-t2i", "price_usd": 0.048 }
      },
      "video": {
        "economy": { "model": "runway-gen4-720p-5s", "price_usd": 0.072 },
        "smart": { "model": "sora-2-10s", "price_usd": 0.18 },
        "best": { "model": "veo-3-1-fast", "price_usd": 0.36 }
      }
    },
    "platform_fee_rate": 0.20,
    "total_models": 271
  }
}
```

---

## Detail Response

```bash
curl https://fairstack.ai/v1/models/seedream-v4-t2i
```

```json
{
  "request_id": "req_def456",
  "data": {
    "slug": "seedream-v4-t2i",
    "name": "Seedream V4",
    "type": "t2i",
    "modality": "image",
    "provider": "runpod",
    "tagline": "High-quality photorealistic image generation",
    "description": "ByteDance's Seedream V4 delivers exceptional photorealistic quality with fine detail rendering...",
    "pricing": {
      "cost_per_request_micro": 20000,
      "cost_per_request_usd": 0.020,
      "total_with_fee_micro": 24000,
      "total_with_fee_usd": 0.024,
      "platform_fee_rate": 0.20
    },
    "capabilities": {
      "key_features": ["Photorealism", "Fine detail", "Accurate lighting"],
      "strengths": ["Product photography", "Landscape", "Architecture"],
      "limitations": ["Slower than turbo models", "Less stylized output"],
      "best_for": ["Commercial photography", "Marketing assets", "Realistic scenes"]
    },
    "intelligence": {
      "scores": {
        "photorealism": 0.92,
        "text_rendering": 0.65,
        "creativity": 0.78,
        "speed": 0.60,
        "consistency": 0.88
      }
    },
    "parameters": {
      "width": { "type": "number", "min": 256, "max": 2048, "default": 1024 },
      "height": { "type": "number", "min": 256, "max": 2048, "default": 768 },
      "aspect_ratio": { "type": "enum", "values": ["1:1", "4:3", "3:4", "16:9", "9:16"] },
      "steps": { "type": "number", "min": 1, "max": 50 },
      "seed": { "type": "number" },
      "negative_prompt": { "type": "string" }
    }
  }
}
```

---

## Query Parameters

### `GET /v1/models`

| Parameter | Type | Description |
|-----------|------|-------------|
| `modality` | string | Filter: `"image"`, `"video"`, `"voice"`, `"music"`, `"talkingHead"`, `"threeD"` |
| `type` | string | Filter by model type within modality (e.g. `"t2i"`, `"edit"`, `"t2v"`, `"i2v"`) |
| `ids` | string | Comma-separated model slugs for batch detail lookup (max 50) |

### `GET /v1/models/:slug`

Returns full detail for a single model including pricing, capabilities, intelligence scores, and supported parameters.

### `GET /v1/models/:slug/params`

Returns the supported parameter schema for a model — types, ranges, enums, and defaults.

---

## Modalities

| Modality | Models | Types |
|----------|--------|-------|
| **image** | 150+ | `t2i` (text-to-image), `edit` (image editing), `i2i` (image-to-image), `upscale`, `utility` |
| **video** | 50+ | `t2v` (text-to-video), `i2v` (image-to-video), `v2v` (video-to-video) |
| **voice** | 30+ | `tts` (text-to-speech), `stt` (speech-to-text) |
| **music** | 10+ | `music` (composition, covers, editing) |
| **talkingHead** | 5+ | `talking-head` (face animation from audio) |
| **threeD** | 5+ | `3d` (3D model generation) |

---

## Popular Models Quick Reference

### Image
| Slug | Cost (w/ fee) | Speed | Quality |
|------|--------------|-------|---------|
| `z-image-turbo` | $0.005 | Fastest | Good |
| `gpt-image-1.5-t2i` | $0.011 | Fast | Great |
| `seedream-v4-t2i` | $0.024 | Medium | Excellent |
| `flux-2-pro` | $0.030 | Medium | Excellent |
| `imagen-4-t2i` | $0.048 | Medium | Best |

### Video
| Slug | Cost (w/ fee) | Duration | Quality |
|------|--------------|----------|---------|
| `runway-gen4-720p-5s` | $0.072 | 5s | Good |
| `kling-v2.5-5s` | $0.096 | 5s | Great |
| `sora-2-10s` | $0.180 | 10s | Excellent |
| `veo-3-t2v` | $0.300 | 10s | Best |

### Voice
| Slug | Cost (w/ fee) | Speed | Quality |
|------|--------------|-------|---------|
| `qwen-3-tts` | ~$0.001 | Fast | Good |
| `chatterbox-turbo` | ~$0.002 | Fast | Great |
| `elevenlabs-turbo-2-5` | ~$0.004 | Fast | Premium |

### Music
| Slug | Cost (w/ fee) | Duration | Quality |
|------|--------------|----------|---------|
| `suno-v4` | ~$0.06 | 30s | Good |
| `ace-step` | ~$0.18 | 30s | Great |
| `mureka-v7.6` | ~$0.60 | 90s | Best |

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 400 | `validation_error` | Invalid `ids` parameter (empty or exceeds 50). |
| 404 | `NOT_FOUND` | Model slug not found. |
| 429 | — | Rate limited. Models endpoint: 60 req/min. |
