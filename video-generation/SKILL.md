---
name: fairstack-video-generation
description: |
  Generate videos via the FairStack API. Text-to-video, image-to-video (first frame),
  first+last frame interpolation, and video-to-video. Models include Sora 2, Kling 2.5,
  Veo 3, Seedance, and Wan 2.1. Async by default — submit and poll. Pricing from
  $0.06/clip. Trigger on: generate video, create video, AI video, text to video,
  image to video, video generation, t2v, i2v, animate image, video clip.
---

# Video Generation

Generate videos from text prompts or images. Sora 2, Kling 2.5, Veo 3, and 50+ more models — all through a single endpoint with transparent pricing.

**Base URL:** `https://fairstack.ai`
**Endpoint:** `POST /v1/video/generate`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Quick Start

Video generation is **async by default** — you submit a job and poll for the result.

### curl

```bash
# 1. Submit video generation
curl -X POST https://fairstack.ai/v1/video/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A drone shot flying over a misty mountain forest at sunrise",
    "model": "sora-2-10s",
    "duration_sec": 10,
    "aspect_ratio": "16:9"
  }'

# Response: {"data": {"generation_id": "gen_vid_789", "status": "running", "poll_url": "/v1/generations/gen_vid_789", ...}}

# 2. Poll for completion
curl https://fairstack.ai/v1/generations/gen_vid_789 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### JavaScript / TypeScript

```typescript
// Submit
const submit = await fetch("https://fairstack.ai/v1/video/generate", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FAIRSTACK_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    prompt: "A drone shot flying over a misty mountain forest at sunrise",
    model: "sora-2-10s",
    duration_sec: 10,
    aspect_ratio: "16:9",
  }),
});

const { data: job } = await submit.json();
console.log(job.generation_id); // gen_vid_789
console.log(job.status);        // "running"

// Poll until complete
async function pollUntilDone(generationId: string): Promise<any> {
  while (true) {
    const res = await fetch(
      `https://fairstack.ai/v1/generations/${generationId}`,
      { headers: { Authorization: `Bearer ${process.env.FAIRSTACK_API_KEY}` } }
    );
    const { data } = await res.json();
    if (data.status === "succeeded") return data;
    if (data.status === "failed") throw new Error(data.error || "Generation failed");
    await new Promise((r) => setTimeout(r, 5000)); // Wait 5s between polls
  }
}

const result = await pollUntilDone(job.generation_id);
console.log(result.url);      // CDN URL to video
console.log(result.cost_usd); // e.g. 0.18
```

### Python

```python
import requests, os, time

headers = {"Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}"}

# Submit
resp = requests.post(
    "https://fairstack.ai/v1/video/generate",
    headers=headers,
    json={
        "prompt": "A drone shot flying over a misty mountain forest at sunrise",
        "model": "sora-2-10s",
        "duration_sec": 10,
        "aspect_ratio": "16:9",
    },
)
job = resp.json()["data"]
gen_id = job["generation_id"]

# Poll
while True:
    status = requests.get(
        f"https://fairstack.ai/v1/generations/{gen_id}",
        headers=headers,
    ).json()["data"]
    if status["status"] == "succeeded":
        print(status["url"])
        break
    if status["status"] == "failed":
        raise Exception(status.get("error", "Failed"))
    time.sleep(5)
```

---

## Submit Response (202)

```json
{
  "request_id": "req_abc123",
  "data": {
    "generation_id": "gen_vid_789",
    "job_id": "gen_vid_789",
    "status": "running",
    "poll_url": "/v1/generations/gen_vid_789",
    "estimated_wait_sec": 60,
    "estimated_cost": 0.18,
    "estimated_cost_micro": 180000,
    "currency": "USD",
    "created_at": "2026-03-23T10:15:00Z"
  }
}
```

## Completed Response

```json
{
  "request_id": "req_def456",
  "data": {
    "id": "gen_vid_789",
    "status": "succeeded",
    "url": "https://media.fairstack.ai/generations/gen_vid_789.mp4",
    "model": "sora-2-10s",
    "duration_sec": 10,
    "cost_micro": 180000,
    "cost_usd": 0.18
  }
}
```

---

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | **Yes** | What to generate (max 25,000 chars) |
| `model` | string | No | Model slug (default from `quality` tier) |
| `quality` | string | No | `"economy"`, `"smart"` (default), `"best"` |
| `duration_sec` | number | No | Video length 3–16 seconds (default 5) |
| `aspect_ratio` | string | No | `"16:9"` (default), `"9:16"`, `"1:1"` |
| `image_url` | string | No | First frame image URL (image-to-video) |
| `last_image_url` | string | No | Last frame image URL (first+last frame interpolation) |
| `video_url` | string | No | Source video URL (video-to-video restyle/extend) |
| `negative_prompt` | string | No | What to avoid in the output |
| `parent_generation_id` | string | No | Link to source generation for edit chains |
| `options` | object | No | Model-specific pass-through parameters |
| `confirm` | boolean | No | Set `false` to get a cost quote without generating |
| `quote_id` | string | No | Confirm a previously created quote |

### Sync Mode

Pass `wait=true` as a query parameter to block until the video is ready (returns 200 with the full result instead of 202):

```bash
curl -X POST "https://fairstack.ai/v1/video/generate?wait=true" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "A cat playing piano", "model": "kling-v2.5-5s"}'
```

### Quality Tiers

| Tier | Model | Cost | Duration |
|------|-------|------|----------|
| `economy` | `runway-gen4-720p-5s` | ~$0.06 | 5s |
| `smart` | `sora-2-10s` | ~$0.15 | 10s |
| `best` | `veo-3-1-fast` | ~$0.30 | 10s |

---

## Models

Popular video models and their pricing:

| Model | Cost | Duration | Best for |
|-------|------|----------|----------|
| `runway-gen4-720p-5s` | ~$0.06 | 5s | Budget-friendly, quick clips |
| `kling-v2.5-5s` | ~$0.08 | 5s | Good quality, image-to-video |
| `sora-2-10s` | ~$0.15 | 10s | Cinematic, long coherent motion |
| `seedance-2-t2v` | ~$0.10 | 5s | Dance, character animation |
| `wan-2.1-t2v` | ~$0.08 | 5s | Anime, stylized content |
| `veo-3-t2v` | ~$0.25 | 10s | Premium quality, Google's best |
| `veo-3-1-fast` | ~$0.30 | 10s | Fastest premium, production-ready |

Use `GET /v1/models?modality=video` for the full catalog of 50+ models with current pricing.

---

## Examples

### Image-to-Video (Animate an Image)

```bash
curl -X POST https://fairstack.ai/v1/video/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "The camera slowly zooms in as leaves rustle in the wind",
    "image_url": "https://example.com/landscape.jpg",
    "model": "kling-v2.5-5s",
    "duration_sec": 5
  }'
```

### First + Last Frame

```bash
curl -X POST https://fairstack.ai/v1/video/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Smooth transition between the two scenes",
    "image_url": "https://example.com/frame-start.jpg",
    "last_image_url": "https://example.com/frame-end.jpg",
    "model": "sora-2-10s",
    "duration_sec": 5
  }'
```

### Cost Quote Before Generating

```bash
curl -X POST https://fairstack.ai/v1/video/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A cinematic sunset over the ocean",
    "model": "veo-3-1-fast",
    "duration_sec": 10,
    "confirm": false
  }'

# Returns quote_id + estimated cost, no credits spent
```

---

## Video Extend

Extend an existing video with a continuation:

```bash
curl -X POST https://fairstack.ai/v1/video/extend \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "source_generation_id": "gen_vid_789",
    "prompt": "The camera continues panning right to reveal a waterfall",
    "duration_sec": 5
  }'
```

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits. Credits are deducted upfront for video. |
| 402 | `QUOTA_EXCEEDED` | Monthly spending cap reached. |
| 422 | `validation_error` | Invalid parameters. |
| 429 | — | Rate limited. Video generation: 600 req/min. |
| 503 | `SERVICE_UNAVAILABLE` | Provider temporarily unavailable. |

---

## Pricing

All prices are **infrastructure cost + 20% platform fee**. No hidden charges.

- Video credits are deducted **upfront** (estimated cost). Reconciled when generation completes.
- If generation fails, credits are automatically refunded.
- Costs returned as `estimated_cost_micro` (integer) and `estimated_cost` (float USD)
- Use `confirm: false` to check cost before committing
