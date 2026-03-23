---
name: fairstack-sfx-library
description: |
  Search and use 9,393 royalty-included sound effects via the FairStack API.
  Browse by category, search by keyword/tags, get random clips, use in projects.
  $0.001 per use. Trigger on: sound effect, sfx, foley, ambient sound, audio clip,
  sound library, background audio, sound design.
---

# Sound Effects Library

Search and use 9,393 sound effects — foley, ambient, UI sounds, and more. Browse by category, search by keyword, or grab a random clip. $0.001 per use.

**Base URL:** `https://fairstack.ai`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Search Sound Effects

### `GET /api/sfx`

Search and filter sound effects with pagination.

```bash
curl "https://fairstack.ai/api/sfx?q=explosion&category=impacts&limit=10" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | No | Search by name or tags |
| `category` | string | No | Filter by category |
| `license` | string | No | Filter by license type |
| `page` | number | No | Page number (default: 1) |
| `limit` | number | No | Results per page (1-100, default: 20) |

### Response

```json
{
  "items": [
    {
      "id": 4521,
      "name": "Explosion Large",
      "url": "data/stock-media/sfx/impacts/freesound-12345.mp3",
      "durationSeconds": 3.2,
      "category": "impacts",
      "tags": ["explosion", "boom", "blast"],
      "format": "mp3",
      "loopable": false,
      "license": "CC0",
      "source": "freesound",
      "attributionRequired": false
    }
  ],
  "total": 142,
  "page": 1,
  "limit": 10,
  "totalPages": 15
}
```

---

## Random Sound Effect

### `GET /api/sfx/random`

Get a random sound effect, optionally filtered by category.

```bash
curl "https://fairstack.ai/api/sfx/random?category=nature" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `category` | string | No | Limit to a specific category |

---

## Categories

### `GET /api/sfx/categories`

List all categories with clip counts.

```bash
curl "https://fairstack.ai/api/sfx/categories" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Response

```json
{
  "categories": [
    { "name": "nature", "count": 1842 },
    { "name": "impacts", "count": 1205 },
    { "name": "ui", "count": 893 },
    { "name": "ambient", "count": 756 }
  ]
}
```

---

## Get Single SFX

### `GET /api/sfx/:id`

```bash
curl "https://fairstack.ai/api/sfx/4521" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## Use a Sound Effect

### `POST /api/sfx/:id/use`

Use a sound effect in your project. Deducts $0.001 (1,000 microdollars) and returns the download URL.

```bash
curl -X POST "https://fairstack.ai/api/sfx/4521/use" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Response

```json
{
  "url": "/media/stock/sfx/impacts/freesound-12345.mp3",
  "license": "CC0",
  "attribution_required": false,
  "attribution_text": "",
  "cost_micro": 1000
}
```

---

## JavaScript / TypeScript

```typescript
import FairStack from "@fairstack/sdk";

const fs = new FairStack({ apiKey: process.env.FAIRSTACK_API_KEY! });

// Search
const results = await fs.sfx.search({ q: "footsteps", category: "foley", limit: 5 });
console.log(`Found ${results.total} sound effects`);

// Random
const random = await fs.sfx.random({ category: "nature" });
console.log(random.name);

// Categories
const cats = await fs.sfx.categories();
console.log(cats.categories);

// Use (deducts $0.001)
const used = await fs.sfx.use(4521);
console.log(used.url); // Download URL
```

## Python

```python
import requests, os

headers = {"Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}"}

# Search
resp = requests.get(
    "https://fairstack.ai/api/sfx",
    headers=headers,
    params={"q": "rain", "limit": 5},
)
sfx = resp.json()["items"]

# Use
resp = requests.post(
    f"https://fairstack.ai/api/sfx/{sfx[0]['id']}/use",
    headers=headers,
)
print(resp.json()["url"])
```

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 400 | — | Invalid SFX ID |
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits ($0.001 needed) |
| 404 | — | SFX not found |
| 429 | — | Rate limited |

---

## Pricing

- **Search/browse:** Free
- **Use a clip:** $0.001 per use (1,000 microdollars)
- **Total library:** 9,393 sound effects across all categories
