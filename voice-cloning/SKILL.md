---
name: fairstack-voice-cloning
description: |
  Clone a voice from reference audio and use it for text-to-speech via the FairStack API.
  Create custom voices, manage cloned voice library, use in TTS generation.
  Trigger on: clone voice, voice clone, custom voice, my voice, replicate voice,
  voice copy, personal voice, clone my voice.
---

# Voice Cloning

Clone any voice from a reference audio sample, then use it for text-to-speech generation. Create up to 10 custom voices per account.

**Base URL:** `https://fairstack.ai`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Create a Voice Clone

### `POST /api/voice-clones`

Create a new cloned voice from reference audio.

```bash
curl -X POST https://fairstack.ai/api/voice-clones \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Custom Voice",
    "description": "Professional narrator voice",
    "referenceAudioUrl": "https://example.com/my-voice-sample.mp3",
    "referenceText": "This is a sample of my voice for cloning purposes.",
    "modelSlug": "f5-tts",
    "isPublic": false
  }'
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | **Yes** | Display name for the cloned voice |
| `description` | string | No | Description of the voice |
| `referenceAudioUrl` | string | **Yes** | URL to a clean audio sample (10-30 seconds recommended) |
| `referenceText` | string | No | Transcript of the reference audio (improves quality) |
| `modelSlug` | string | **Yes** | TTS model to use (`f5-tts`, `kokoro-tts`, etc.) |
| `sampleUrl` | string | No | URL to a preview/sample clip |
| `isPublic` | boolean | No | Share with community (default: false) |
| `creatorDisplayName` | string | No | Public display name when sharing |

### Response (201)

```json
{
  "clone": {
    "id": "vc_abc123",
    "name": "My Custom Voice",
    "description": "Professional narrator voice",
    "referenceAudioUrl": "https://example.com/my-voice-sample.mp3",
    "modelSlug": "f5-tts",
    "isPublic": false,
    "createdAt": "2026-03-23T10:00:00Z"
  }
}
```

---

## List Your Cloned Voices

### `GET /api/voice-clones`

```bash
curl https://fairstack.ai/api/voice-clones \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Response

```json
{
  "clones": [
    {
      "id": "vc_abc123",
      "name": "My Custom Voice",
      "description": "Professional narrator voice",
      "referenceAudioUrl": "https://example.com/my-voice-sample.mp3",
      "modelSlug": "f5-tts",
      "isPublic": false,
      "createdAt": "2026-03-23T10:00:00Z"
    }
  ]
}
```

---

## Get a Cloned Voice

### `GET /api/voice-clones/:id`

```bash
curl https://fairstack.ai/api/voice-clones/vc_abc123 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## Update a Clone

### `PATCH /api/voice-clones/:id`

```bash
curl -X PATCH https://fairstack.ai/api/voice-clones/vc_abc123 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Updated Voice Name", "isPublic": true}'
```

---

## Delete a Clone

### `DELETE /api/voice-clones/:id`

```bash
curl -X DELETE https://fairstack.ai/api/voice-clones/vc_abc123 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## Use a Cloned Voice for TTS

Once created, use your cloned voice in any TTS generation via the `ref_audio_url` parameter:

```bash
curl -X POST https://fairstack.ai/v1/voice/generate \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Hello, this is my cloned voice speaking!",
    "model": "f5-tts",
    "ref_audio_url": "https://example.com/my-voice-sample.mp3",
    "ref_text": "This is a sample of my voice for cloning purposes."
  }'
```

---

## JavaScript / TypeScript

```typescript
import FairStack from "@fairstack/sdk";

const fs = new FairStack({ apiKey: process.env.FAIRSTACK_API_KEY! });

// Create a clone
const clone = await fs.voiceClones.create({
  name: "My Voice",
  referenceAudioUrl: "https://example.com/voice-sample.mp3",
  referenceText: "Sample transcript for better quality.",
  modelSlug: "f5-tts",
});

// List all clones
const clones = await fs.voiceClones.list();
console.log(`${clones.clones.length} cloned voices`);

// Use cloned voice for generation
const speech = await fs.voice.generate({
  text: "This is my cloned voice!",
  model: "f5-tts",
  ref_audio_url: clone.clone.referenceAudioUrl,
  ref_text: clone.clone.referenceText,
});
console.log(speech.output?.url);

// Delete a clone
await fs.voiceClones.delete(clone.clone.id);
```

## Python

```python
import requests, os

headers = {
    "Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}",
    "Content-Type": "application/json",
}

# Create clone
resp = requests.post(
    "https://fairstack.ai/api/voice-clones",
    headers=headers,
    json={
        "name": "My Voice",
        "referenceAudioUrl": "https://example.com/sample.mp3",
        "modelSlug": "f5-tts",
    },
)
clone = resp.json()["clone"]

# Use in generation
resp = requests.post(
    "https://fairstack.ai/v1/voice/generate",
    headers=headers,
    json={
        "text": "Hello from my cloned voice!",
        "model": "f5-tts",
        "ref_audio_url": clone["referenceAudioUrl"],
    },
)
print(resp.json()["output"]["url"])
```

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 400 | — | Clone limit reached (10 max) or invalid input |
| 404 | — | Clone not found or not owned by you |
| 429 | — | Rate limited |

---

## Pricing

- **Creating clones:** Free (no credit charge)
- **Using cloned voice for TTS:** Standard voice generation pricing ($0.001+)
- **Clone limit:** 10 per account (free tier)
