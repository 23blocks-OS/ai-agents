---
name: 23blocks-search-entity-search-api
description: "Search Block structured entity search: filters, nested JSONB queries, sorting, optional vectors. Use for precise filtered search."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Entity Search API

Complete API reference for 23blocks advanced entity search with AI vector search support.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://search.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### POST /entities/search - Search Entities

Performs advanced entity search with include/exclude filters, nested JSONB field queries, pagination, sorting, and optional AI-powered vector similarity search.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "search": {
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
    }
  }'
```

**Request Parameters** (nested inside `"search": { ... }`)**:**
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "search": {
      "include": {
        "entity_type": "article",
        "content.status": "published"
      }
    }
  }'
```

### Nested JSONB Field Query
```bash
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "search": {
      "include": {
        "content.author.name": "John Doe",
        "content.tags": ["technology", "ai"]
      }
    }
  }'
```

### Range Filters
```bash
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "search": {
      "include": {
        "entity_type": "product",
        "content.price": { "min": 10, "max": 50 },
        "content.rating": { "min": 4 }
      },
      "sort_by": "content.rating",
      "sort_order": "desc"
    }
  }'
```

### AI Vector Search
```bash
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "search": {
      "ai_search": true,
      "ai_query": "sustainable eco-friendly products for home office",
      "include": {
        "entity_type": "product"
      },
      "records": 10
    }
  }'
```

### Combined Include and Exclude
```bash
curl -X POST "$BLOCKS_API_URL/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "search": {
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
    }
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"400","code":"invalid_filter","title":"Invalid Filter","detail":"The filter structure for 'include' is invalid."}]}`.
