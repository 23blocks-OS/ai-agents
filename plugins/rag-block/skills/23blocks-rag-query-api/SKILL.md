---
name: 23blocks-rag-query-api
description: Query RAG-processed files using natural language semantic search. Use when searching product catalogs, performing visual search with object detection, or querying documents by meaning rather than keywords.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# RAG Query API

Query processed files using natural language semantic search. Supports text queries, visual search with automatic multi-object detection, score thresholds, and reranking.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | RAG API base URL | `https://jarvis.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://jarvis.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://jarvis.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### POST /products/:product_id/query - Semantic Search

Queries processed files using natural language. Returns ranked results based on vector similarity.

**Resource scoping:** Replace `/products/:product_id` with the appropriate resource path:
- `/products/:id` — Product catalog files
- `/accounts/:id` — CRM account files
- `/contacts/:id` — CRM contact files
- `/users/:id` — User profile files
- `/storage/:company_url` — Company storage files

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/$PRODUCT_ID/query" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "red wine from Bordeaux region",
    "limit": 10,
    "score_threshold": 0.7,
    "use_reranking": false
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | string | Yes | — | Natural language search query |
| `limit` | integer | No | 10 | Maximum number of results to return |
| `score_threshold` | float | No | 0.7 | Minimum similarity score (0.0-1.0) |
| `use_reranking` | boolean | No | false | Enable result reranking for improved relevance |
| `use_object_detection` | boolean | No | true | Enable multi-object detection for visual search |

**Response 200:**
```json
{
  "data": [
    {
      "type": "search_result",
      "attributes": {
        "chunk_id": "chunk_abc",
        "file_unique_id": "file_xyz",
        "content": "Chateau Margaux 2018 Bordeaux...",
        "score": 0.92,
        "metadata": {
          "source_file": "wine_catalog.pdf",
          "page": 12
        }
      }
    }
  ],
  "meta": {
    "total_results": 5,
    "query": "red wine from Bordeaux region",
    "processing_time_ms": 250
  }
}
```

---

### POST /products/search - Visual Search

Search products using text or image queries with automatic multi-object detection.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "Pinot Noir bottle",
    "use_object_detection": true,
    "limit": 10,
    "score_threshold": 0.7
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | string | Yes | — | Text or image-based search query |
| `use_object_detection` | boolean | No | true | Enable multi-object detection for image queries |
| `limit` | integer | No | 10 | Maximum number of results |
| `score_threshold` | float | No | 0.7 | Minimum similarity score (0.0-1.0) |

**Response 200 (with object detection):**
```json
{
  "data": [
    {
      "type": "search_result",
      "attributes": {
        "product_id": "prod_abc",
        "name": "Chateau Margaux 2018",
        "score": 0.94,
        "detection_metadata": {
          "detection": "multi_object",
          "object_index": 0,
          "total_objects": 3,
          "class": "bottle",
          "confidence": 0.91
        }
      }
    }
  ],
  "meta": {
    "objects_detected": 3,
    "total_results": 8,
    "processing_time_ms": 1800
  }
}
```

---

## Object Detection in Search

`use_object_detection` defaults to `true`. Every visual search automatically runs object detection:

| Objects Detected | Behavior |
|-----------------|----------|
| 0-1 objects | Standard single search (automatic fallback) |
| 2+ objects | Multi-object search — each object searched individually, results grouped |

**Performance:**
- Object detection: ~500ms (YOLOv8)
- CLIP embedding per object: ~200ms
- Total for 5 objects: ~1.5-2.5 seconds

Set `use_object_detection: false` to skip detection entirely and use standard search.

---

## Data Models

### SearchResult

| Field | Type | Description |
|-------|------|-------------|
| `chunk_id` | string | Unique identifier for the matched chunk |
| `file_unique_id` | string | Source file ID |
| `content` | string | Matched text content |
| `score` | float | Similarity score (0.0-1.0) |
| `metadata` | object | Source metadata (file name, page, etc.) |
| `detection_metadata` | object | Object detection details (when applicable) |

### SearchMeta

| Field | Type | Description |
|-------|------|-------------|
| `total_results` | integer | Total number of results returned |
| `objects_detected` | integer | Number of objects detected (visual search only) |
| `query` | string | Original query string |
| `processing_time_ms` | integer | Total processing time in milliseconds |

---

## Use Cases

### Text Document Search
```bash
# Search product documentation
curl -X POST "$BLOCKS_API_URL/products/$PRODUCT_ID/query" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "query": "return policy for damaged items" }'
```

### Multi-Object Visual Search (Shelf Photo)
```bash
# Search all products in a shelf photo
curl -X POST "$BLOCKS_API_URL/products/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "identify all bottles",
    "use_object_detection": true
  }'
```

### Inventory Agent Workflow
1. User uploads shelf photo
2. Process with `object_detection` mode via ingestion API
3. Response: `objects_detected: 5`
4. Agent: "I found 5 bottles. Create 5 product entries?"
5. User confirms
6. Agent creates products using Products API

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "title": "Not Found",
    "detail": "Product not found."
  }]
}
```

Common status codes: `401` Invalid API key, `404` Product/file not found, `422` Validation error, `500` Internal server error, `503` Service unavailable.
