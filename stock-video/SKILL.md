---
name: fairstack-stock-video
description: |
  Search and use 18,172 royalty-included B-roll video clips via the FairStack API.
  Clips from Pexels/Pixabay with category, aspect ratio, pacing, and motion filters.
  $0.002 per use. Trigger on: b-roll, stock video, stock footage, video clip,
  background video, stock media, footage library.
---

# Stock Video / B-Roll Library

Search and use 18,172 B-roll video clips from Pexels and Pixabay. Filter by category, aspect ratio, pacing, and motion intensity. $0.002 per use.

**Base URL:** `https://fairstack.ai`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Search B-Roll Clips

### `GET /api/broll`

Search and filter B-roll clips with pagination.

```bash
curl "https://fairstack.ai/api/broll?q=sunset&category=nature&aspect_ratio=16:9&limit=10" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | No | Search by name, tags, and AI keywords |
| `category` | string | No | Filter by category |
| `aspect_ratio` | string | No | `16:9`, `9:16`, `1:1`, `4:5` |
| `pacing` | string | No | `slow`, `medium`, `fast` |
| `motion_intensity` | string | No | `static`, `subtle`, `dynamic` |
| `page` | number | No | Page number (default: 1) |
| `limit` | number | No | Results per page (1-100, default: 20) |

### Response

```json
{
  "items": [
    {
      "id": "abc123-def456",
      "name": "Golden sunset over ocean",
      "category": "nature",
      "tags": ["sunset", "ocean", "golden hour"],
      "clipUrl": "https://videos.pexels.com/...",
      "thumbnailUrl": "https://images.pexels.com/...",
      "duration": 12.5,
      "width": 1920,
      "height": 1080,
      "license": "pexels",
      "source": "pexels",
      "aspectRatio": "16:9",
      "pacing": "slow",
      "motionIntensity": "subtle",
      "aiKeywords": ["sunset", "water", "horizon", "calm"],
      "attributionRequired": false
    }
  ],
  "total": 847,
  "page": 1,
  "limit": 10,
  "totalPages": 85
}
```

---

## Random Clip

### `GET /api/broll/random`

Get a random B-roll clip, optionally filtered by category.

```bash
curl "https://fairstack.ai/api/broll/random?category=technology" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `category` | string | No | Limit to a specific category |

---

## Categories

### `GET /api/broll/categories`

List all categories with clip counts.

```bash
curl "https://fairstack.ai/api/broll/categories" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Response

```json
{
  "categories": [
    { "name": "nature", "count": 4210 },
    { "name": "city", "count": 2843 },
    { "name": "technology", "count": 1956 },
    { "name": "people", "count": 1724 }
  ]
}
```

---

## Get Single Clip

### `GET /api/broll/:id`

```bash
curl "https://fairstack.ai/api/broll/abc123-def456" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## Use a Clip

### `POST /api/broll/:id/use`

Use a B-roll clip in your project. Deducts $0.002 (2,000 microdollars) and returns the download URL with metadata.

```bash
curl -X POST "https://fairstack.ai/api/broll/abc123-def456/use" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Response

```json
{
  "url": "https://videos.pexels.com/...",
  "thumbnail_url": "https://images.pexels.com/...",
  "duration": 12.5,
  "width": 1920,
  "height": 1080,
  "license": "pexels",
  "attribution_required": false,
  "source": "pexels",
  "cost_micro": 2000
}
```

---

## JavaScript / TypeScript

```typescript
import FairStack from "@fairstack/sdk";

const fs = new FairStack({ apiKey: process.env.FAIRSTACK_API_KEY! });

// Search
const results = await fs.broll.search({
  q: "aerial city",
  aspect_ratio: "16:9",
  pacing: "slow",
  limit: 5,
});
console.log(`Found ${results.total} clips`);

// Random
const random = await fs.broll.random({ category: "nature" });
console.log(random.name);

// Categories
const cats = await fs.broll.categories();
console.log(cats.categories);

// Use (deducts $0.002)
const used = await fs.broll.use("abc123-def456");
console.log(used.url); // Download URL
```

## Python

```python
import requests, os

headers = {"Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}"}

# Search
resp = requests.get(
    "https://fairstack.ai/api/broll",
    headers=headers,
    params={"q": "sunset", "aspect_ratio": "16:9", "limit": 5},
)
clips = resp.json()["items"]

# Use
resp = requests.post(
    f"https://fairstack.ai/api/broll/{clips[0]['id']}/use",
    headers=headers,
)
print(resp.json()["url"])
```

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 400 | — | Invalid clip ID |
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits ($0.002 needed) |
| 404 | — | Clip not found |
| 429 | — | Rate limited |

---

## Pricing

- **Search/browse:** Free
- **Use a clip:** $0.002 per use (2,000 microdollars)
- **Total library:** 18,172 clips from Pexels and Pixabay
