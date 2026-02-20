---
name: search-jarvis-search-api
description: Execute AI-powered natural language entity search via REST API. Use when translating user queries in plain language to structured search using OpenAI ChatGPT for intelligent entity discovery.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Jarvis AI Search API

Complete API reference for 23blocks AI-powered natural language entity search using OpenAI ChatGPT.

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
curl -X POST "$BLOCKS_API_URL/jarvis/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /jarvis/entities/search - AI Natural Language Search

Translates a natural language query into a structured entity search using OpenAI ChatGPT. The AI analyzes the user's intent, extracts search parameters, and executes the appropriate filtered search on the entity index.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/jarvis/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "Find all electronics under $50 with high ratings",
    "entity_type": "product"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Natural language search query |
| `entity_type` | string | No | Scope search to specific entity type |
| `language` | string | No | Query language (auto-detected if omitted) |
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

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
        "entity_alias": "Budget Wireless Earbuds",
        "content": {
          "name": "Budget Wireless Earbuds",
          "category": "electronics",
          "price": 24.99,
          "rating": 4.5
        },
        "status": "active"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 18,
    "interpreted_query": {
      "include": {
        "entity_type": "product",
        "content.category": "electronics",
        "content.price": { "max": 50 },
        "content.rating": { "min": 4 }
      }
    },
    "original_query": "Find all electronics under $50 with high ratings",
    "elapsed_time": 1.23
  }
}
```

**Errors:**
- `400 Bad Request` - Empty or invalid query
- `422 Unprocessable Entity` - AI could not interpret query
- `503 Service Unavailable` - OpenAI service unavailable

---

## Usage Examples

### Simple Product Search
```bash
curl -X POST "$BLOCKS_API_URL/jarvis/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "show me the latest smartphones",
    "entity_type": "product"
  }'
```

### Multi-criteria Search
```bash
curl -X POST "$BLOCKS_API_URL/jarvis/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "articles about machine learning published this year by senior authors"
  }'
```

### Location-based Search
```bash
curl -X POST "$BLOCKS_API_URL/jarvis/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "restaurants near downtown with vegan options and outdoor seating",
    "entity_type": "business"
  }'
```

### Multi-language Search
```bash
curl -X POST "$BLOCKS_API_URL/jarvis/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "buscar productos de tecnologia baratos",
    "language": "es"
  }'
```

---

## How It Works

1. **User submits** a natural language query
2. **ChatGPT analyzes** the query to extract intent, filters, and sort criteria
3. **Query translator** converts the AI response into structured search parameters
4. **Entity search** executes with the translated include/exclude filters
5. **Results returned** with both the matched entities and the interpreted query

## AI Capabilities

| Feature | Description |
|---------|-------------|
| **Intent Detection** | Understands what the user is looking for |
| **Filter Extraction** | Extracts numeric ranges, categories, dates |
| **Sort Inference** | Determines appropriate sorting from context |
| **Multi-language** | Accepts queries in multiple languages |
| **Entity Scoping** | Automatically narrows to relevant entity types |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "query_interpretation_error",
    "title": "Could Not Interpret Query",
    "detail": "The AI could not translate the query into structured search parameters."
  }]
}
```
