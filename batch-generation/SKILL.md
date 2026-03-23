---
name: fairstack-batch-generation
description: |
  Generate at scale with the FairStack Batch API. Run multiple generations in
  a single request — prompt sweeps (one model, many prompts), model sweeps
  (one prompt, many models), and parameter sweeps (vary one param). Up to 10
  items per batch across image, voice, video, and music. Trigger on: batch
  generate, generate at scale, bulk images, multiple images, prompt sweep,
  model comparison, batch generation, parallel generation.
---

# Batch Generation

Run up to 10 generations in a single API call. Compare models, sweep prompts, or vary parameters — all with one request.

**Base URL:** `https://fairstack.ai`
**Endpoint:** `POST /v1/batch`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Three Batch Modes

| Mode | Description | Use case |
|------|-------------|----------|
| `prompt_sweep` | One model, multiple prompts | Generate a series of images from different descriptions |
| `model_sweep` | One prompt, multiple models | Compare quality across models |
| `param_sweep` | One prompt + model, vary a parameter | Test different aspect ratios, seeds, or styles |

---

## Quick Start

### Prompt Sweep — Same Model, Multiple Prompts

```bash
curl -X POST https://fairstack.ai/v1/batch \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "prompt_sweep",
    "type": "image",
    "model": "gpt-image-1.5-t2i",
    "prompts": [
      "A red sports car on a coastal highway",
      "A vintage motorcycle in a desert at sunset",
      "A sailboat in rough ocean waves"
    ]
  }'
```

### Model Sweep — Same Prompt, Multiple Models

```bash
curl -X POST https://fairstack.ai/v1/batch \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "model_sweep",
    "type": "image",
    "prompt": "A photorealistic portrait of an astronaut on Mars",
    "models": [
      "z-image-turbo",
      "gpt-image-1.5-t2i",
      "seedream-v4-t2i",
      "imagen-4-t2i"
    ]
  }'
```

### Parameter Sweep — Vary One Parameter

```bash
curl -X POST https://fairstack.ai/v1/batch \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "param_sweep",
    "type": "image",
    "model": "flux-2-pro",
    "prompt": "A majestic mountain landscape",
    "sweep_param": "aspect_ratio",
    "sweep_values": ["1:1", "16:9", "9:16", "21:9"]
  }'
```

---

## JavaScript / TypeScript

```typescript
const response = await fetch("https://fairstack.ai/v1/batch", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FAIRSTACK_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    mode: "prompt_sweep",
    type: "image",
    model: "gpt-image-1.5-t2i",
    prompts: [
      "A cyberpunk cityscape at night",
      "A serene Japanese garden in autumn",
      "An underwater coral reef teeming with life",
    ],
  }),
});

const { data } = await response.json();
// data.results — array of generation results
// data.total_cost_usd — total cost for the batch
for (const result of data.results) {
  console.log(result.url, result.cost_usd);
}
```

## Python

```python
import requests, os

resp = requests.post(
    "https://fairstack.ai/v1/batch",
    headers={"Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}"},
    json={
        "mode": "model_sweep",
        "type": "image",
        "prompt": "A golden sunset over the Sahara desert",
        "models": ["z-image-turbo", "gpt-image-1.5-t2i", "imagen-4-t2i"],
    },
)

data = resp.json()["data"]
for result in data["results"]:
    print(f"{result['model']}: {result['url']} (${result['cost_usd']})")
```

---

## Parameters

### Prompt Sweep

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mode` | `"prompt_sweep"` | **Yes** | |
| `type` | string | **Yes** | `"image"`, `"voice"`, `"video"`, or `"music"` |
| `model` | string | **Yes** | Model slug to use for all prompts |
| `prompts` | string[] | **Yes** | Array of prompts (1–10 items) |
| `options` | object | No | Shared options for all generations |

### Model Sweep

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mode` | `"model_sweep"` | **Yes** | |
| `type` | string | **Yes** | Modality (all models must match this) |
| `prompt` | string | **Yes** | Shared prompt for all models |
| `models` | string[] | **Yes** | Array of model slugs (1–10 items) |
| `options` | object | No | Shared options for all generations |

### Parameter Sweep

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mode` | `"param_sweep"` | **Yes** | |
| `type` | string | **Yes** | Modality |
| `model` | string | **Yes** | Model slug |
| `prompt` | string | **Yes** | Shared prompt |
| `sweep_param` | string | **Yes** | Parameter name to vary |
| `sweep_values` | any[] | **Yes** | Values to try (1–10 items) |
| `options` | object | No | Base options (sweep_param overrides) |

---

## Response

```json
{
  "request_id": "req_batch_789",
  "data": {
    "batch_id": "batch_abc123",
    "results": [
      {
        "index": 0,
        "id": "gen_001",
        "status": "succeeded",
        "url": "https://media.fairstack.ai/generations/gen_001.png",
        "model": "gpt-image-1.5-t2i",
        "cost_micro": 10800,
        "cost_usd": 0.0108
      },
      {
        "index": 1,
        "id": "gen_002",
        "status": "succeeded",
        "url": "https://media.fairstack.ai/generations/gen_002.png",
        "model": "gpt-image-1.5-t2i",
        "cost_micro": 10800,
        "cost_usd": 0.0108
      }
    ],
    "total_cost_micro": 21600,
    "total_cost_usd": 0.0216
  }
}
```

---

## Limits

- **Max 10 items** per batch request
- Credit check is **upfront** — total estimated cost is verified before any generation starts
- Failed individual items do not block others — each result has its own status
- Batch supports all modalities: image, voice, video, music

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits for the entire batch. |
| 422 | `validation_error` | Invalid parameters, unknown model, or exceeds max batch size. |
| 429 | — | Rate limited. 600 req/min. |

---

## Pricing

Batch pricing is the **sum of individual generation costs**. No batch surcharge.

- Each item in the batch costs the same as an individual generation
- All prices are infrastructure cost + 20% platform fee
- Use `POST /v1/estimate` to check per-model costs before batching
