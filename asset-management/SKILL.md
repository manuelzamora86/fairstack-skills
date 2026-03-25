---
name: fairstack-asset-management
description: |
  Manage your generated and uploaded assets via the FairStack API. List, filter,
  tag, search, download, and delete assets. Bulk delete with R2 cleanup. Auto-tagging
  adds AI-powered semantic labels to every generation. Trigger on: assets, my files,
  my generations, list assets, delete asset, bulk delete, manage files, asset library,
  find assets, asset tags, organize assets.
---

# Asset Management

Manage all your generated and uploaded files — list, filter, tag, search, download, and bulk-delete. Every generation is automatically saved as an asset with AI-powered tags.

**Base URL:** `https://fairstack.ai`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Quick Start

### CLI

```bash
# List recent assets
fairstack assets list --limit 10

# Filter by modality
fairstack assets list --modality image --limit 20

# Find assets by tag
fairstack assets list --tag "hero-image"

# Delete an asset
fairstack assets delete asset_abc123

# Bulk delete
fairstack assets delete asset_001 asset_002 asset_003

# List tags on an asset
fairstack tags list asset_abc123

# Add tags to an asset
fairstack tags add asset_abc123 --tags "marketing, hero, homepage"

# Search assets by tag
fairstack tags search --tag "album-cover"
```

---

## List Assets

### `GET /v1/assets`

List and filter your assets with pagination.

```bash
# List all assets
curl "https://fairstack.ai/v1/assets?limit=20" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"

# Filter by modality
curl "https://fairstack.ai/v1/assets?modality=image&limit=10" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"

# Filter by tag
curl "https://fairstack.ai/v1/assets?tag=hero-image&limit=10" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"

# Filter by tag key + value
curl "https://fairstack.ai/v1/assets?tagKey=style&tagValue=cinematic" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `modality` | string | No | `"image"`, `"video"`, `"voice"`, `"music"` |
| `source` | string | No | `"generation"`, `"upload"` |
| `tag` | string | No | Filter by tag value (matches any key) |
| `tagKey` | string | No | Filter by tag key |
| `tagValue` | string | No | Filter by tag value (use with tagKey) |
| `limit` | number | No | Results per page (1-100, default: 20) |
| `offset` | number | No | Pagination offset (default: 0) |

### Response

```json
{
  "items": [
    {
      "id": "asset_abc123",
      "url": "https://media.fairstack.ai/generations/gen_xyz789.png",
      "modality": "image",
      "source": "generation",
      "title": "Mountain sunset",
      "model": "seedream-v4-t2i",
      "tags": [
        { "key": "auto:1", "value": "landscape" },
        { "key": "auto:2", "value": "sunset" },
        { "key": "user:1", "value": "hero-image" }
      ],
      "favorite": false,
      "created_at": "2026-03-23T10:15:00Z"
    }
  ],
  "total": 142,
  "limit": 20,
  "offset": 0,
  "has_more": true
}
```

---

## Get Asset

### `GET /v1/assets/:id`

```bash
curl https://fairstack.ai/v1/assets/asset_abc123 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## Upload Assets

### Upload from URL

#### `POST /v1/assets/upload-url`

```bash
curl -X POST https://fairstack.ai/v1/assets/upload-url \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/my-image.png",
    "modality": "image",
    "title": "Uploaded reference image"
  }'
```

### Upload File

#### `POST /v1/assets/upload`

```bash
curl -X POST https://fairstack.ai/v1/assets/upload \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -F "file=@my-image.png"
```

---

## Update Asset

### `PATCH /v1/assets/:id`

Update title, favorite status, or tags.

```bash
curl -X PATCH https://fairstack.ai/v1/assets/asset_abc123 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"title": "Hero image for homepage", "favorite": true}'
```

---

## Delete Asset

### `DELETE /v1/assets/:id`

Soft-deletes an asset. Permanently removed from R2 after 30 days.

```bash
curl -X DELETE https://fairstack.ai/v1/assets/asset_abc123 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Bulk Delete

#### `DELETE /v1/assets/bulk`

Delete up to 100 assets at once. Validates org ownership for every ID.

```bash
curl -X DELETE https://fairstack.ai/v1/assets/bulk \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"ids": ["asset_001", "asset_002", "asset_003"]}'
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `ids` | string[] | **Yes** | Asset IDs to delete (max 100) |

#### Response

```json
{
  "data": {
    "deleted": 3,
    "ids": ["asset_001", "asset_002", "asset_003"]
  }
}
```

---

## Tag Management

Every generation is automatically tagged with AI-powered semantic labels. You can also add your own tags.

### List Tags

#### `GET /v1/assets/:id/tags`

```bash
curl https://fairstack.ai/v1/assets/asset_abc123/tags \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

#### Response

```json
[
  { "key": "auto:1", "value": "landscape" },
  { "key": "auto:2", "value": "sunset" },
  { "key": "auto:3", "value": "mountain" },
  { "key": "user:1", "value": "hero-image" },
  { "key": "user:2", "value": "marketing" }
]
```

Tags with `auto:` prefix are AI-generated. Tags with `user:` prefix are user-added.

### Add Tags

#### `POST /v1/assets/:id/tags`

```bash
curl -X POST https://fairstack.ai/v1/assets/asset_abc123/tags \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"tags": ["marketing", "hero", "homepage"]}'
```

#### Response

```json
[
  { "key": "user:3", "value": "marketing" },
  { "key": "user:4", "value": "hero" },
  { "key": "user:5", "value": "homepage" }
]
```

### Remove a Tag

#### `DELETE /v1/assets/:id/tags/:key`

```bash
curl -X DELETE https://fairstack.ai/v1/assets/asset_abc123/tags/user%3A3 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## Download

### `GET /v1/assets/:id/download`

Get a signed download URL for an asset.

```bash
curl https://fairstack.ai/v1/assets/asset_abc123/download \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## JavaScript / TypeScript

```typescript
import FairStack from "@fairstack/sdk";

const fs = new FairStack({ apiKey: process.env.FAIRSTACK_API_KEY! });

// List image assets
const assets = await fs.assets.list({ modality: "image", limit: 20 });
console.log(`${assets.total} image assets`);

// Filter by tag
const heroes = await fs.assets.list({ tag: "hero-image" });

// Upload from URL
const uploaded = await fs.assets.uploadUrl({
  url: "https://example.com/reference.png",
  modality: "image",
  title: "Reference image",
});

// List tags
const tags = await fs.assets.listTags("asset_abc123");
for (const tag of tags) {
  console.log(`${tag.key}: ${tag.value}`);
}

// Add tags
await fs.assets.addTags("asset_abc123", ["marketing", "hero", "homepage"]);

// Remove a tag
await fs.assets.removeTag("asset_abc123", "user:1");

// Delete
await fs.assets.delete("asset_abc123");

// Download
const download = await fs.assets.download("asset_abc123");
console.log(download.url); // Signed URL
```

## Python

```python
import requests, os

headers = {
    "Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}",
    "Content-Type": "application/json",
}

# List assets filtered by tag
resp = requests.get(
    "https://fairstack.ai/v1/assets",
    headers=headers,
    params={"tag": "hero-image", "limit": 10},
)
assets = resp.json()["items"]
print(f"{len(assets)} assets with 'hero-image' tag")

# List tags on an asset
resp = requests.get(
    "https://fairstack.ai/v1/assets/asset_abc123/tags",
    headers=headers,
)
for tag in resp.json():
    print(f"  {tag['key']}: {tag['value']}")

# Add tags
requests.post(
    "https://fairstack.ai/v1/assets/asset_abc123/tags",
    headers=headers,
    json={"tags": ["marketing", "hero"]},
)

# Bulk delete
requests.delete(
    "https://fairstack.ai/v1/assets/bulk",
    headers=headers,
    json={"ids": ["asset_001", "asset_002", "asset_003"]},
)
```

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 400 | `validation_error` | Invalid parameters (e.g., more than 100 IDs for bulk delete) |
| 404 | — | Asset not found or not owned by your org |
| 429 | — | Rate limited |

---

## Best Practices

### Tag everything at generation time

Include `tags` in every generation request. Auto-tagging adds AI labels, but your own project-specific tags make assets findable later.

```bash
--tags "hero-image, marketing, homepage"
```

### Use structured tags for filtering

For complex projects, use key:value tags instead of plain values:

```json
{
  "tags": [
    {"key": "campaign", "value": "q1-launch"},
    {"key": "style", "value": "cinematic"},
    {"key": "usage", "value": "social-media"}
  ]
}
```

### Clean up regularly

Unused assets accumulate storage. Use bulk delete to clean up test generations and old iterations.

---

## Pricing

- **Asset management:** Free (listing, tagging, organizing)
- **Storage:** Included in generation cost (no extra fees)
- **Soft delete -> permanent delete:** 30 days after soft delete, files are removed from R2
