<p align="center">
  <img src="https://fairstack.ai/favicon.svg" width="48" height="48" alt="FairStack" />
</p>

<h1 align="center">FairStack Skills</h1>

<p align="center">
  <strong>AI generation at fair prices. Images, voice, video, and music — through one API.</strong>
</p>

<p align="center">
  <a href="https://fairstack.ai">Website</a> ·
  <a href="https://fairstack.ai/docs">Docs</a> ·
  <a href="https://fairstack.ai/app/api-keys">Get API Key</a> ·
  <a href="https://fairstack.ai/pricing">Pricing</a>
</p>

---

## What is FairStack?

FairStack is the **Cloudflare of AI generation** — 270+ models across image, voice, video, and music, all with transparent pricing. We charge infrastructure cost + 20% margin. That's it. No subscriptions, no tiers, no hidden fees.

## Skills

Each skill is a self-contained guide for one capability of the FairStack API:

### Generation

| Skill | Description | Endpoint |
|-------|-------------|----------|
| [**Image Generation**](./image-generation/SKILL.md) | Text-to-image, image editing, 150+ models from $0.004 | `POST /v1/image/generate` |
| [**Voice Generation**](./voice-generation/SKILL.md) | Text-to-speech, 168 voices, voice cloning, emotions | `POST /v1/voice/generate` |
| [**Video Generation**](./video-generation/SKILL.md) | Text-to-video, image-to-video, Sora/Kling/Veo | `POST /v1/video/generate` |
| [**Music Generation**](./music-generation/SKILL.md) | AI composition with lyrics, BPM, key controls | `POST /v1/music/generate` |
| [**Talking Head**](./talking-head/SKILL.md) | Lip-synced avatar video from text + image ($0.35 vs $11) | `POST /v1/dialogue/generate` |
| [**AI Dubbing**](./dubbing/SKILL.md) | Translate video/audio to another language | `POST /v1/dub/generate` |

### Media Libraries

| Skill | Description | Endpoint |
|-------|-------------|----------|
| [**Sound Effects**](./sfx-library/SKILL.md) | 9,393 SFX — foley, ambient, UI sounds ($0.001/use) | `GET /api/sfx` |
| [**Stock Video**](./stock-video/SKILL.md) | 18,172 B-roll clips from Pexels/Pixabay ($0.002/use) | `GET /api/broll` |

### Advanced

| Skill | Description | Endpoint |
|-------|-------------|----------|
| [**Model Compare**](./model-compare/SKILL.md) | Side-by-side comparison across 2-4 models | `POST /v1/{modality}/compare` |
| [**Workflows**](./workflows/SKILL.md) | Multi-step generation pipelines with SSE streaming | `POST /v1/workflows` |
| [**Voice Cloning**](./voice-cloning/SKILL.md) | Clone a voice from reference audio, use for TTS | `POST /api/voice-clones` |
| [**Batch Generation**](./batch-generation/SKILL.md) | Up to 10 generations in one call — sweeps & comparisons | `POST /v1/batch` |

### Utilities

| Skill | Description | Endpoint |
|-------|-------------|----------|
| [**Cost Estimation**](./cost-estimate/SKILL.md) | Check prices, create quotes, enhance prompts (free) | `POST /v1/estimate` |
| [**Smart Model Selection**](./smart-select/SKILL.md) | AI-powered model recommendation with reasoning | `POST /v1/select-model` |
| [**Model Catalog**](./models/SKILL.md) | Browse 270+ models, filter by modality, get details | `GET /v1/models` |

## Quick Start

### 1. Get an API Key

Sign up at [fairstack.ai](https://fairstack.ai) and create an API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

### 2. Set Your Key

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

### 3. Generate

```bash
# Image — $0.009
curl -X POST https://fairstack.ai/v1/image/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "A mountain lake at sunset", "model": "gpt-image-1.5-t2i"}'

# Voice — $0.001
curl -X POST https://fairstack.ai/v1/voice/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello world", "voice": "internal_marco"}'

# Video — $0.15 (async)
curl -X POST https://fairstack.ai/v1/video/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "A drone shot over mountains", "model": "sora-2-10s"}'

# Music — $0.50 (async)
curl -X POST https://fairstack.ai/v1/music/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Upbeat indie pop", "duration_sec": 60}'

# Sound effects — $0.001
curl "https://fairstack.ai/api/sfx?q=explosion" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"

# Stock video — $0.002
curl "https://fairstack.ai/api/broll?q=sunset&aspect_ratio=16:9" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

## Authentication

Every request needs an `Authorization` header:

```
Authorization: Bearer fs_live_...
```

Exception: `GET /v1/models` is public (no auth required).

## Pricing

**Infrastructure cost + 20% platform fee.** No subscriptions. No tiers. No hidden charges.

| What you generate | Starting at |
|-------------------|-------------|
| Image | $0.005 |
| Voice (per request) | $0.001 |
| Video (5s clip) | $0.07 |
| Music (30s) | $0.06 |
| Talking head video | $0.35 |
| Sound effect (per use) | $0.001 |
| Stock video clip (per use) | $0.002 |

Every response includes exact cost in both microdollars (`cost_micro`) and USD (`cost_usd`). Use [`POST /v1/estimate`](./cost-estimate/SKILL.md) to check cost before generating.

## Error Codes

| Status | Code | What to do |
|--------|------|------------|
| 402 | `INSUFFICIENT_CREDITS` | Add credits at the `fund_url` in the response |
| 402 | `QUOTA_EXCEEDED` | Monthly spending cap reached — adjust in settings |
| 422 | `validation_error` | Check the `message` field for what's wrong |
| 429 | — | Rate limited (600 req/min for generation, 60 req/min for other) |
| 503 | `SERVICE_UNAVAILABLE` | Provider down — try a different model or retry later |

## Async Generation Pattern

Video and music are async by default. Image and voice are sync by default.

```
                    ┌─────────────┐
                    │  Submit Job │
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              │                         │
        ┌─────┴─────┐           ┌──────┴──────┐
        │  Sync      │           │  Async       │
        │  (image,   │           │  (video,     │
        │   voice)   │           │   music)     │
        └─────┬─────┘           └──────┬──────┘
              │                         │
              │ 200 + result            │ 202 + generation_id
              │                         │
              │                    ┌────┴────┐
              │                    │  Poll    │
              │                    │  GET /v1/│
              │                    │  genera- │
              │                    │  tions/  │
              │                    │  :id     │
              │                    └────┬────┘
              │                         │
              └─────────┬───────────────┘
                        │
                   ┌────┴────┐
                   │  Result  │
                   │  + URL   │
                   └─────────┘
```

Override defaults: add `?wait=false` to image/voice for async, `?wait=true` to video/music for sync.

## Links

- **Website:** [fairstack.ai](https://fairstack.ai)
- **API Docs:** [fairstack.ai/docs](https://fairstack.ai/docs)
- **Dashboard:** [fairstack.ai/app](https://fairstack.ai/app)
- **Status:** [fairstack.ai/status](https://fairstack.ai/status)
- **OpenAPI Spec:** [fairstack.ai/v1/openapi.json](https://fairstack.ai/v1/openapi.json)
