---
name: search-search-api
description: Execute 23blocks full-text search queries via REST API. Use when performing text searches on the MasterIndex or retrieving recent search query history.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Search API

Complete API reference for 23blocks full-text search on the MasterIndex.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://search.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Search API base URL | `https://search.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X POST "$BLOCKS_API_URL/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /search - Execute Search Query

Executes a full-text search query on the MasterIndex. The search supports narrow (AND) and wide (OR) modes and automatically logs the query with performance metrics.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "machine learning algorithms",
    "page": 1,
    "records": 20
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Search query text |
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `search_type` | string | No | Search mode: "narrow" (AND) or "wide" (OR) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "result-uuid-123",
      "type": "search_result",
      "attributes": {
        "key": "entity-key-123",
        "content": "Matched content from the MasterIndex...",
        "score": 0.95
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 87,
    "query": "machine learning algorithms",
    "search_type": "narrow",
    "elapsed_time": 0.045
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Missing or empty query

---

### GET /search/:id - Get Recent Search Queries

Retrieves the most recent search queries for the current user. The `:id` parameter is used as the limit (number of queries to return).

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/search/10" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | integer | Yes | Number of recent queries to return |

**Response 200:**
```json
{
  "data": [
    {
      "id": "query-uuid-123",
      "type": "last_query",
      "attributes": {
        "query": "machine learning algorithms",
        "user_uid": "user-uuid-123",
        "submitted_at": "2025-01-12T10:30:00Z"
      }
    },
    {
      "id": "query-uuid-124",
      "type": "last_query",
      "attributes": {
        "query": "neural networks",
        "user_uid": "user-uuid-123",
        "submitted_at": "2025-01-12T10:25:00Z"
      }
    }
  ]
}
```

---

## Search Modes

| Mode | Behavior | Example |
|------|----------|---------|
| `narrow` (AND) | All terms must match | "machine learning" matches only records containing both words |
| `wide` (OR) | Any term can match | "machine learning" matches records containing either word |
| default | Platform default mode | Depends on configuration |

## Data Models

### Query (Search Log)
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Query identifier |
| `query` | string | Search query text |
| `query_hash` | string | SHA256 hash for deduplication |
| `include` | object | Include filters applied |
| `exclude` | object | Exclude filters applied |
| `search_type` | string | narrow, wide, or default |
| `total_records` | integer | Number of results found |
| `elapsed_time` | float | Query execution time in seconds |
| `user_unique_id` | uuid | User who performed search |
| `started_at` | timestamp | Query start time |
| `ended_at` | timestamp | Query end time |

### LastQuery
| Field | Type | Description |
|-------|------|-------------|
| `query` | string | Search query text |
| `user_uid` | uuid | User who submitted query |
| `submitted_at` | timestamp | When query was submitted |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Invalid Query",
    "detail": "Search query cannot be empty."
  }]
}
```
