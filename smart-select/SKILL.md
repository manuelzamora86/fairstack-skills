---
name: fairstack-smart-select
description: |
  AI-powered model recommendation via the FairStack API. Describe what you
  want to create and get the best model recommendation with reasoning and
  alternatives. Uses Claude + model intelligence data. Supports priority
  preferences (quality, speed, cost). Trigger on: which model, best model,
  recommend model, pick a model, select model, model recommendation,
  what model should I use, smart select, model advisor.
---

# Smart Model Selection

Don't know which model to use? Describe what you want and let AI pick the best one. Powered by Claude + intelligence data from 270+ models.

**Base URL:** `https://fairstack.ai`
**Endpoint:** `POST /v1/select-model`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

> **Note:** This endpoint also works without authentication — unauthenticated requests get a helpful signup prompt alongside the recommendation.

---

## Quick Start

### curl

```bash
curl -X POST https://fairstack.ai/v1/select-model \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "I need to generate a photorealistic product photo of a sneaker on a white background",
    "modality": "image",
    "priority": "quality",
    "project": "shoe-catalog",
    "tags": [
      {"key": "tag", "value": "product-photo"},
      {"key": "tag", "value": "sneaker"},
      {"key": "tag", "value": "model-selection"}
    ]
  }'
```

### JavaScript / TypeScript

```typescript
const response = await fetch("https://fairstack.ai/v1/select-model", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FAIRSTACK_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    prompt: "I need a cinematic 10-second video of a rocket launch",
    modality: "video",
    priority: "quality",
  }),
});

const { data } = await response.json();
console.log(data.recommended_model); // e.g. "veo-3-t2v"
console.log(data.reasoning);         // Why this model was chosen
console.log(data.alternatives);      // Other options with tradeoffs

// Use the recommendation directly
const generateResponse = await fetch("https://fairstack.ai/v1/video/generate", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FAIRSTACK_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    prompt: "A cinematic rocket launch with dramatic lighting",
    model: data.recommended_model,
    duration_sec: 10,
  }),
});
```

### Python

```python
import requests, os

resp = requests.post(
    "https://fairstack.ai/v1/select-model",
    headers={"Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}"},
    json={
        "prompt": "Generate background music for a meditation app",
        "modality": "music",
        "priority": "quality",
    },
)

data = resp.json()["data"]
print(f"Use: {data['recommended_model']}")
print(f"Why: {data['reasoning']}")
print(f"Cost: ${data['estimated_cost_usd']}")
```

---

## Response

```json
{
  "request_id": "req_abc123",
  "data": {
    "recommended_model": "seedream-v4-t2i",
    "modality": "image",
    "reasoning": "Seedream V4 excels at photorealistic product photography with clean backgrounds. It handles commercial product shots with accurate lighting and material rendering better than general-purpose models.",
    "estimated_cost_usd": 0.024,
    "estimated_cost_micro": 24000,
    "alternatives": [
      {
        "model": "imagen-4-t2i",
        "reasoning": "Higher quality but 2x cost. Best if budget allows.",
        "estimated_cost_usd": 0.048
      },
      {
        "model": "gpt-image-1.5-t2i",
        "reasoning": "Good text rendering if you need brand names on the product. 60% cheaper.",
        "estimated_cost_usd": 0.0108
      }
    ],
    "intelligence_version": "2026-03-20"
  }
}
```

---

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | **Yes** | Describe what you want to create |
| `modality` | string | No | `"image"`, `"video"`, `"voice"`, `"music"`. Auto-detected if omitted. |
| `priority` | string | No | `"quality"` (default), `"speed"`, or `"cost"` |
| `budget_max_usd` | number | No | Maximum cost per generation in USD |
| `exclude_models` | string[] | No | Models to exclude from recommendations |

---

## How It Works

1. Your prompt is analyzed against intelligence data from 270+ models
2. Each model has scored capabilities (photorealism, text rendering, motion coherence, etc.)
3. Claude matches your needs to model strengths, weighted by your priority
4. Returns the best match + 2-3 alternatives with tradeoff explanations

The intelligence data includes:
- **Quality scores** per task dimension (0.0–1.0)
- **Speed ratings** (fast/medium/slow)
- **Known issues** and limitations
- **Best-for** use cases
- **Price-performance ratio**

---

## Examples

### Budget-Constrained Selection

```bash
curl -X POST https://fairstack.ai/v1/select-model \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Generate 100 product thumbnails for an e-commerce store",
    "modality": "image",
    "priority": "cost",
    "budget_max_usd": 0.01
  }'
```

### Auto-Detect Modality

```bash
# Don't specify modality — let the API figure it out
curl -X POST https://fairstack.ai/v1/select-model \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "I need a 30-second lo-fi hip hop beat for a YouTube video"
  }'
# Automatically detects: modality=music
```

### Speed Priority

```bash
curl -X POST https://fairstack.ai/v1/select-model \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Quick avatar generation for user profiles",
    "modality": "image",
    "priority": "speed"
  }'
```

---

## Best Practices

### Use project + tags with select-model

Tag your model selection requests so you can trace back which model was recommended for each project.

### Style consistency

Smart select is the starting point for locking in your project's model. After getting a recommendation:

1. **Record the recommended model in your project's style guide.** Don't re-query for every generation -- use the same model consistently.
2. **Re-select only when your needs change** (e.g., switching from quality to cost priority, or a new model launches).
3. **For agents:** Call select-model once at project setup, record the result, and reference it in all subsequent generation calls.

---

## Pricing

Smart model selection costs a small fee (Claude API token cost + 10% margin). Typically **$0.001–$0.005 per recommendation** depending on prompt length.

The cost is automatically deducted from your credit balance and included in the response.

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits for the recommendation. |
| 422 | `validation_error` | Invalid parameters. |
| 429 | — | Rate limited. |
| 503 | `SERVICE_UNAVAILABLE` | Recommendation service temporarily unavailable. |
