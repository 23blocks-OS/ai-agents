---
name: search-entity-search-api
description: Execute advanced 23blocks entity search via REST API. Use when performing filtered entity search with include/exclude criteria, nested JSONB queries, pagination, sorting, and optional AI-powered vector search.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Entity Search API

Complete API reference for 23blocks advanced entity search with AI vector search support.

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
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /entities/search - Search Entities

Performs advanced entity search with include/exclude filters, nested JSONB field queries, pagination, sorting, and optional AI-powered vector similarity search.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "include": {
      "entity_type": "product",
      "content.category": "electronics",
      "content.price": { "min": 10, "max": 100 }
    },
    "exclude": {
      "status": "inactive"
    },
    "page": 1,
    "records": 20,
    "sort_by": "content.price",
    "sort_order": "asc"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `include` | object | No | Fields and values to include in results |
| `exclude` | object | No | Fields and values to exclude from results |
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `sort_by` | string | No | Field to sort by (supports nested: content.field) |
| `sort_order` | string | No | asc or desc (default: desc) |
| `entity_type` | string | No | Filter by specific entity type |
| `ai_search` | boolean | No | Enable AI vector similarity search |
| `ai_query` | string | No | Natural language query for vector search |

**Response 200:**
```json
{
  "data": [
    {
      "id": "entity-uuid-123",
      "type": "entity_identity",
      "attributes": {
        "unique_id": "entity-uuid-123",
        "entity_type": "product",
        "entity_alias": "Widget Pro",
        "slug": "widget-pro",
        "content": {
          "name": "Widget Pro",
          "category": "electronics",
          "price": 29.99,
          "description": "Advanced widget with AI features"
        },
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 45,
    "elapsed_time": 0.032
  }
}
```

**Errors:**
- `400 Bad Request` - Invalid filter structure
- `422 Unprocessable Entity` - Validation errors

---

## Filter Examples

### Basic Include Filter
```bash
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "include": {
      "entity_type": "article",
      "content.status": "published"
    }
  }'
```

### Nested JSONB Field Query
```bash
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "include": {
      "content.author.name": "John Doe",
      "content.tags": ["technology", "ai"]
    }
  }'
```

### Range Filters
```bash
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "include": {
      "entity_type": "product",
      "content.price": { "min": 10, "max": 50 },
      "content.rating": { "min": 4 }
    },
    "sort_by": "content.rating",
    "sort_order": "desc"
  }'
```

### AI Vector Search
```bash
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "ai_search": true,
    "ai_query": "sustainable eco-friendly products for home office",
    "include": {
      "entity_type": "product"
    },
    "records": 10
  }'
```

### Combined Include and Exclude
```bash
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "include": {
      "entity_type": "product",
      "content.category": "electronics"
    },
    "exclude": {
      "content.brand": "generic",
      "status": "inactive"
    },
    "page": 1,
    "records": 25
  }'
```

---

## Search Capabilities

| Feature | Description |
|---------|-------------|
| **Include Filters** | Match entities where fields contain specified values |
| **Exclude Filters** | Exclude entities where fields match specified values |
| **Nested Fields** | Query deep JSONB paths (e.g., `content.author.name`) |
| **Range Queries** | Filter by min/max on numeric and date fields |
| **Array Matching** | Match against array values with OR logic |
| **Text Search** | Accent-insensitive text matching via unaccent |
| **AI Vector** | Semantic similarity using OpenAI embeddings + pgvector |
| **Pagination** | Page-based with configurable page size |
| **Sorting** | Dynamic sorting on any field including nested |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "400",
    "code": "invalid_filter",
    "title": "Invalid Filter",
    "detail": "The filter structure for 'include' is invalid."
  }]
}
```
