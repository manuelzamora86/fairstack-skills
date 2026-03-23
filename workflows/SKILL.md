---
name: fairstack-workflows
description: |
  Create and run multi-step generation pipelines via the FairStack API.
  Chain image, voice, video, and music generations into automated workflows.
  SSE streaming for real-time progress. Trigger on: workflow, pipeline, chain,
  multi-step, automate, sequence, generation pipeline, orchestrate.
---

# Workflows

Create and run multi-step generation pipelines. Chain image, voice, video, and music generations into automated workflows with real-time SSE streaming progress.

**Base URL:** `https://fairstack.ai`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

---

## Create a Workflow

### `POST /v1/workflows`

Define a workflow as a graph of nodes and edges.

```bash
curl -X POST https://fairstack.ai/v1/workflows \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Image to Video Pipeline",
    "description": "Generate an image, then animate it to video",
    "graph": {
      "nodes": [
        {
          "id": "img1",
          "type": "image_generate",
          "data": {
            "prompt": "{{subject}} in a cinematic style",
            "model": "gpt-image-1.5-t2i"
          }
        },
        {
          "id": "vid1",
          "type": "video_generate",
          "data": {
            "prompt": "Slow zoom into the scene",
            "model": "kling-v2-5s",
            "image_url": "{{img1.output.url}}"
          }
        }
      ],
      "edges": [
        { "id": "e1", "source": "img1", "sourceHandle": "output", "target": "vid1", "targetHandle": "image_url" }
      ]
    }
  }'
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | **Yes** | Workflow name (max 200 chars) |
| `description` | string | No | Description (max 2,000 chars) |
| `graph` | object | **Yes** | Workflow graph with `nodes` and `edges` arrays |

### Response (201)

```json
{
  "data": {
    "workflow": {
      "id": "wf_abc123",
      "name": "Image to Video Pipeline",
      "description": "Generate an image, then animate it to video",
      "graph": { "nodes": [...], "edges": [...] },
      "is_template": false,
      "is_public": false,
      "run_count": 0,
      "last_run_at": null,
      "created_at": "2026-03-23T10:00:00Z",
      "updated_at": "2026-03-23T10:00:00Z"
    }
  }
}
```

---

## List Workflows

### `GET /v1/workflows`

```bash
curl "https://fairstack.ai/v1/workflows?limit=10&offset=0" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `limit` | number | No | Results per page (1-100, default: 20) |
| `offset` | number | No | Offset for pagination (default: 0) |

---

## Get a Workflow

### `GET /v1/workflows/:id`

```bash
curl https://fairstack.ai/v1/workflows/wf_abc123 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## Delete a Workflow

### `DELETE /v1/workflows/:id`

```bash
curl -X DELETE https://fairstack.ai/v1/workflows/wf_abc123 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## Run a Workflow (SSE Stream)

### `POST /v1/workflows/:id/run`

Execute a workflow and receive real-time progress via Server-Sent Events.

```bash
curl -N -X POST https://fairstack.ai/v1/workflows/wf_abc123/run \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "variables": {
      "subject": "a majestic lion on a cliff"
    }
  }'
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `variables` | object | No | Key-value variables to inject into node templates |

### SSE Events

```
event: node_started
data: {"nodeId": "img1", "nodeType": "image_generate"}

event: node_completed
data: {"nodeId": "img1", "output": {"url": "https://..."}, "cost_micro": 9000}

event: node_started
data: {"nodeId": "vid1", "nodeType": "video_generate"}

event: node_completed
data: {"nodeId": "vid1", "output": {"url": "https://..."}, "cost_micro": 150000}

event: workflow_completed
data: {"status": "completed", "total_cost_micro": 159000}
```

---

## Execution History

### `GET /v1/workflows/:id/runs`

```bash
curl "https://fairstack.ai/v1/workflows/wf_abc123/runs?limit=5" \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### Response

```json
{
  "data": {
    "runs": [
      {
        "id": "run_xyz789",
        "workflow_id": "wf_abc123",
        "status": "completed",
        "node_results": { "img1": {...}, "vid1": {...} },
        "total_cost_microdollars": 159000,
        "started_at": "2026-03-23T10:05:00Z",
        "completed_at": "2026-03-23T10:07:30Z"
      }
    ],
    "pagination": { "total": 12, "limit": 5, "offset": 0, "has_more": true }
  }
}
```

---

## JavaScript / TypeScript

```typescript
import FairStack from "@fairstack/sdk";

const fs = new FairStack({ apiKey: process.env.FAIRSTACK_API_KEY! });

// Create workflow
const wf = await fs.workflows.create({
  name: "Text to Video Pipeline",
  graph: {
    nodes: [
      { id: "img", type: "image_generate", data: { prompt: "{{subject}}", model: "gpt-image-1.5-t2i" } },
      { id: "vid", type: "video_generate", data: { prompt: "Zoom in", model: "kling-v2-5s" } },
    ],
    edges: [
      { id: "e1", source: "img", sourceHandle: "output", target: "vid", targetHandle: "image_url" },
    ],
  },
});

// Run workflow (returns when complete)
const run = await fs.workflows.run(wf.workflow.id, {
  variables: { subject: "a futuristic city at night" },
});
console.log(`Cost: $${run.total_cost_microdollars / 1_000_000}`);

// List workflows
const list = await fs.workflows.list({ limit: 10 });
console.log(`${list.pagination.total} workflows`);

// Delete
await fs.workflows.delete(wf.workflow.id);
```

## Python

```python
import requests, os

headers = {
    "Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}",
    "Content-Type": "application/json",
}

# Create
resp = requests.post(
    "https://fairstack.ai/v1/workflows",
    headers=headers,
    json={
        "name": "My Pipeline",
        "graph": {
            "nodes": [{"id": "img", "type": "image_generate", "data": {"prompt": "A sunset"}}],
            "edges": [],
        },
    },
)
wf = resp.json()["data"]["workflow"]

# Run (SSE stream)
resp = requests.post(
    f"https://fairstack.ai/v1/workflows/{wf['id']}/run",
    headers=headers,
    json={"variables": {}},
    stream=True,
)
for line in resp.iter_lines():
    if line:
        print(line.decode())
```

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 402 | `INSUFFICIENT_CREDITS` | No credits to run workflow |
| 404 | — | Workflow not found |
| 422 | `validation_error` | Invalid graph structure or empty nodes |
| 429 | — | Rate limited |

---

## Pricing

- **Creating/managing workflows:** Free
- **Running workflows:** Sum of all node generation costs
- Each node charges its standard generation rate
- Real-time cost tracking via SSE events
