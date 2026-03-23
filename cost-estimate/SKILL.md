---
name: fairstack-cost-estimate
description: |
  Estimate generation costs before spending credits via the FairStack API.
  Check prices for image, voice, video, and music generation. Create confirmable
  quotes with 60-second TTL. Free prompt enhancement included. Trigger on:
  how much, estimate cost, price check, what would it cost, cost estimate,
  check price, generation cost, credit check, enhance prompt.
---

# Cost Estimation

Check the cost of any generation before spending credits. Create confirmable quotes, enhance prompts for free, and always know what you'll pay.

**Base URL:** `https://fairstack.ai`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Cost Estimate

### `POST /v1/estimate`

Get the exact cost for a generation without creating one.

```bash
curl -X POST https://fairstack.ai/v1/estimate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "modality": "image",
    "model": "imagen-4-t2i"
  }'
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `modality` | string | **Yes** | `"image"`, `"voice"`, `"video"`, or `"music"` |
| `model` | string | No | Specific model slug. Uses default if omitted. |
| `text` | string | No | Text input (for voice cost estimation based on length) |
| `duration_sec` | number | No | Duration in seconds (for video/music) |
| `width` | number | No | Image width (affects cost for some models) |
| `height` | number | No | Image height |
| `aspect_ratio` | string | No | Aspect ratio (video) |
| `options` | object | No | Model-specific options that may affect pricing |
| `create_quote` | boolean | No | Create a confirmable quote (60s TTL) |

### Response

```json
{
  "request_id": "req_abc123",
  "data": {
    "modality": "image",
    "model": "imagen-4-t2i",
    "estimated_cost": {
      "infra_micro": 40000,
      "margin_micro": 8000,
      "total_micro": 48000,
      "total_usd": 0.048,
      "currency": "USD"
    },
    "margin_rate": 0.20
  }
}
```

---

## Quote-Before-Generate Flow

For expensive generations, use the quote flow to confirm cost before committing:

### Step 1: Get a Quote

Pass `confirm: false` to any generation endpoint:

```bash
curl -X POST https://fairstack.ai/v1/video/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Cinematic aerial shot of a volcano erupting",
    "model": "veo-3-1-fast",
    "duration_sec": 10,
    "confirm": false
  }'
```

Response:

```json
{
  "data": {
    "quote_id": "quote_abc123",
    "estimated_cost": {
      "cost": 0.36,
      "cost_micro": 360000,
      "currency": "USD"
    },
    "expires_at": "2026-03-23T10:16:00Z",
    "expires_in_sec": 60
  }
}
```

### Step 2: Confirm the Quote

```bash
curl -X POST https://fairstack.ai/v1/video/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Cinematic aerial shot of a volcano erupting",
    "model": "veo-3-1-fast",
    "duration_sec": 10,
    "quote_id": "quote_abc123"
  }'
```

Quotes expire after **60 seconds**. If expired, create a new one.

---

## Prompt Enhancement

### `POST /v1/enhance-prompt`

Improve your prompt before generating — **free** (FairStack absorbs the cost).

```bash
curl -X POST https://fairstack.ai/v1/enhance-prompt \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "a cat on a roof",
    "style": "photorealistic"
  }'
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | **Yes** | Original prompt (max 2,000 chars) |
| `style` | string | No | Target style (e.g. `"photorealistic"`, `"anime"`, `"oil painting"`) |
| `negative_prompt` | string | No | What to avoid (max 500 chars) |

### Response

```json
{
  "data": {
    "original": "a cat on a roof",
    "enhanced": "A sleek tabby cat perched on a terracotta tile roof at golden hour, warm sunset light casting long shadows, photorealistic detail, shallow depth of field with bokeh background of a Mediterranean village",
    "changes": "Added specific cat breed, roof material, lighting conditions, camera technique, and setting for more detailed generation"
  }
}
```

---

## JavaScript / TypeScript

```typescript
// Estimate cost
const estimate = await fetch("https://fairstack.ai/v1/estimate", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FAIRSTACK_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    modality: "video",
    model: "sora-2-10s",
    duration_sec: 10,
  }),
});

const { data } = await estimate.json();
console.log(`Estimated cost: $${data.estimated_cost.total_usd}`);

// Enhance prompt (free)
const enhanced = await fetch("https://fairstack.ai/v1/enhance-prompt", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FAIRSTACK_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    prompt: "mountains at sunset",
    style: "cinematic",
  }),
});

const { data: enhancedData } = await enhanced.json();
console.log(enhancedData.enhanced); // Use this for generation
```

## Python

```python
import requests, os

headers = {"Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}"}

# Estimate
resp = requests.post(
    "https://fairstack.ai/v1/estimate",
    headers=headers,
    json={"modality": "image", "model": "seedream-v4-t2i"},
)
cost = resp.json()["data"]["estimated_cost"]
print(f"Image will cost: ${cost['total_usd']}")

# Enhance prompt (free)
resp = requests.post(
    "https://fairstack.ai/v1/enhance-prompt",
    headers=headers,
    json={"prompt": "a dog playing", "style": "watercolor"},
)
print(resp.json()["data"]["enhanced"])
```

---

## Check Your Balance

```bash
curl https://fairstack.ai/v1/account/balance \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

```json
{
  "data": {
    "balance_micro": 5000000,
    "balance_usd": 5.00,
    "currency": "USD"
  }
}
```

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 422 | `validation_error` | Invalid modality, bad parameters. |
| 429 | — | Rate limited. |
| 503 | `SERVICE_UNAVAILABLE` | Prompt enhancement temporarily unavailable. |

---

## Pricing Reference

All FairStack pricing follows one rule: **infrastructure cost + 20% platform fee**.

| Modality | Cheapest Model | Cost | Premium Model | Cost |
|----------|---------------|------|---------------|------|
| Image | `z-image-turbo` | $0.004 | `imagen-4-t2i` | $0.048 |
| Voice | `qwen-3-tts` | ~$0.001 | `minimax-speech-2-8-hd` | ~$0.005 |
| Video | `runway-gen4-720p-5s` | ~$0.07 | `veo-3-1-fast` | ~$0.36 |
| Music | `suno-v4` | ~$0.06 | `mureka-v7.6` | ~$0.60 |

Prices include the 20% platform fee. Use `POST /v1/estimate` for exact current pricing.
