---
name: fairstack-image-generation
description: |
  Generate images via the FairStack API. Text-to-image, image-to-image editing,
  upscaling, background removal, and search-grounded generation. 150+ image models
  from $0.004/image. Supports style presets, aspect ratios, negative prompts, and
  seed reproducibility. Trigger on: generate image, create image, AI image, text to
  image, edit image, image generation, t2i, i2i, upscale image, remove background.
---

# Image Generation

Generate images with 150+ models through a single API. Text-to-image, image editing, upscaling, and background removal — all with transparent per-generation pricing.

**Base URL:** `https://fairstack.ai`
**Endpoint:** `POST /v1/image/generate`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

All requests require `Authorization: Bearer <API_KEY>` header.

---

## Quick Start

### curl

```bash
curl -X POST https://fairstack.ai/v1/image/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A serene mountain lake at sunset, oil painting style",
    "model": "gpt-image-1.5-t2i",
    "aspect_ratio": "16:9",
    "project": "jamwise",
    "tags": [
      {"key": "tag", "value": "hero-image"},
      {"key": "tag", "value": "marketing"},
      {"key": "tag", "value": "landscape"}
    ]
  }'
```

### CLI

```bash
fairstack generate image \
  --prompt "A serene mountain lake at sunset, oil painting style" \
  --model gpt-image-1.5-t2i \
  --aspect-ratio 16:9 \
  --project jamwise \
  --tags "hero-image, marketing, landscape"
```

### JavaScript / TypeScript

```typescript
const response = await fetch("https://fairstack.ai/v1/image/generate", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FAIRSTACK_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    prompt: "A serene mountain lake at sunset, oil painting style",
    model: "gpt-image-1.5-t2i",
    aspect_ratio: "16:9",
    project: "jamwise",
    tags: [
      { key: "tag", value: "hero-image" },
      { key: "tag", value: "marketing" },
      { key: "tag", value: "landscape" },
    ],
  }),
});

const { data } = await response.json();
console.log(data.url);    // CDN URL to generated image
console.log(data.cost_usd); // e.g. 0.0108
```

### Python

```python
import requests, os

resp = requests.post(
    "https://fairstack.ai/v1/image/generate",
    headers={"Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}"},
    json={
        "prompt": "A serene mountain lake at sunset, oil painting style",
        "model": "gpt-image-1.5-t2i",
        "aspect_ratio": "16:9",
        "project": "jamwise",
        "tags": [
            {"key": "tag", "value": "hero-image"},
            {"key": "tag", "value": "marketing"},
            {"key": "tag", "value": "landscape"},
        ],
    },
)

data = resp.json()["data"]
print(data["url"])       # CDN URL
print(data["cost_usd"])  # e.g. 0.0108
```

---

## Response

```json
{
  "request_id": "req_abc123",
  "data": {
    "id": "gen_xyz789",
    "url": "https://media.fairstack.ai/generations/gen_xyz789.png",
    "model": "gpt-image-1.5-t2i",
    "width": 1024,
    "height": 576,
    "cost_micro": 10800,
    "cost_usd": 0.0108,
    "created_at": "2026-03-23T10:15:00Z"
  }
}
```

---

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | **Yes** | What to generate (max 25,000 chars) |
| `model` | string | No | Model slug (see Models below). Default: resolved from `quality` |
| `quality` | string | No | Quality tier: `"economy"`, `"smart"` (default), `"best"` |
| `width` | number | No | Output width 256–2048 (default 1024) |
| `height` | number | No | Output height 256–2048 (default 768) |
| `aspect_ratio` | string | No | `"1:1"`, `"4:3"`, `"3:4"`, `"3:2"`, `"2:3"`, `"16:9"`, `"9:16"`, `"21:9"` |
| `negative_prompt` | string | No | What to avoid in the output |
| `seed` | number | No | Reproducibility seed |
| `image_url` | string | No | Source image URL for editing / image-to-image |
| `image_urls` | string[] | No | Multiple reference images (max 10) |
| `style_id` | string | No | Style preset ID (use `GET /v1/styles` to list) |
| `steps` | number | No | Inference steps 1–50 |
| `options` | object | No | Model-specific pass-through parameters |
| `search_prompt` | string | No | Auto-find reference images from the web |
| `search_sources` | string[] | No | Search providers: `"google"`, `"wikimedia"` |
| `search_count` | number | No | How many search results to consider (1–10) |
| `project` | string | No | Project slug or ID to associate with this generation |
| `tags` | array | No | Key-value tags for organization (max 20). Format: `[{"key":"tag","value":"hero"}]` |
| `confirm` | boolean | No | Set `false` to get a cost quote without generating |
| `quote_id` | string | No | Confirm a previously created quote |

### Quality Tiers

| Tier | Model | Cost | Best for |
|------|-------|------|----------|
| `economy` | `z-image-turbo` | $0.004 | Prototyping, bulk generation, testing |
| `smart` | `gpt-image-1.5-t2i` | $0.009 | Balanced quality and cost (default) |
| `best` | `imagen-4-t2i` | $0.040 | Maximum quality, production assets |

---

## Models

Popular image models and their pricing:

| Model | Cost | Provider | Best for |
|-------|------|----------|----------|
| `z-image-turbo` | $0.004 | Kie.ai | Fastest, cheapest — great for iteration |
| `gpt-image-1.5-t2i` | $0.009 | fal.ai | Strong text rendering, instruction following |
| `seedream-v4-t2i` | ~$0.020 | RunPod | Photorealistic, high detail |
| `flux-2-pro` | ~$0.025 | fal.ai | Artistic, creative compositions |
| `nano-banana-2-t2i` | ~$0.008 | Kie.ai | Fast, good quality balance |
| `imagen-4-t2i` | $0.040 | Kie.ai | Google's premium — best quality |

Use `GET /v1/models?modality=image` for the full catalog of 150+ models with pricing.

---

## Examples

### Image-to-Image Editing

```bash
curl -X POST https://fairstack.ai/v1/image/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Make the sky more dramatic with storm clouds",
    "model": "gpt-image-1.5-t2i",
    "image_url": "https://example.com/landscape.jpg"
  }'
```

### With Style Preset

```bash
# List available styles first
curl https://fairstack.ai/v1/styles

# Generate with a style
curl -X POST https://fairstack.ai/v1/image/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A futuristic city skyline",
    "style_id": "cyberpunk",
    "aspect_ratio": "21:9"
  }'
```

### Cost Quote Before Generating

```bash
# Get a quote (no credits spent)
curl -X POST https://fairstack.ai/v1/image/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "An astronaut riding a horse",
    "model": "imagen-4-t2i",
    "confirm": false
  }'

# Response includes quote_id — use it to confirm:
curl -X POST https://fairstack.ai/v1/image/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "An astronaut riding a horse",
    "model": "imagen-4-t2i",
    "quote_id": "quote_abc123"
  }'
```

### Async Mode

```bash
# Request async processing (returns 202 with poll URL)
curl -X POST "https://fairstack.ai/v1/image/generate?wait=false" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "A detailed fantasy map", "model": "imagen-4-t2i"}'

# Poll for result
curl https://fairstack.ai/v1/generations/gen_xyz789 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits. Response includes `fund_url` to add credits. |
| 402 | `QUOTA_EXCEEDED` | Monthly spending cap reached. |
| 422 | `validation_error` | Invalid parameters (missing prompt, bad dimensions, etc.) |
| 429 | — | Rate limited. Image generation: 600 req/min. |
| 503 | `SERVICE_UNAVAILABLE` | Provider temporarily unavailable. Retry or use a different model. |

### Example Error Response

```json
{
  "request_id": "req_abc123",
  "error": {
    "code": "INSUFFICIENT_CREDITS",
    "message": "Insufficient credits. Estimated cost: $0.0480, balance: $0.0100",
    "fund_url": "https://fairstack.ai/quick-start"
  }
}
```

---

## Best Practices

### Always use project + tags

Every generation should include `project` and at least 3 tags. Tags help you recover, search, and filter your work later.

```bash
--project my-app --tags "hero-image, marketing, homepage"
```

### Style consistency

This is the #1 quality issue users hit. Inconsistent styles across generations make your project look unprofessional.

1. **Define 2-3 reusable style prompts per project.** Save them with the Styles API (`POST /v1/styles`) so every generation in the project uses the same visual language (e.g., "clean vector, flat colors, music-themed").
2. **Use the same model for visual consistency within a project.** Mixing models produces noticeably different aesthetics.
3. **Tag which style was used:** `--tags "marketing-style, hero-image, jamwise"` so you can filter by style later.
4. **For agents:** Create a style guide file that your agent references before generating. Include the model slug, style prompt suffix, and negative prompt.

---

## Pricing

All prices are **infrastructure cost + 20% platform fee**. No hidden charges.

- Costs returned in every response as `cost_micro` (integer microdollars) and `cost_usd` (float)
- 1 USD = 1,000,000 microdollars
- Use `POST /v1/estimate` or `confirm: false` to check cost before generating
