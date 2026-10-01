---
name: 23blocks-search-jarvis-search-api
description: "Search Block natural-language entity search translated by an LLM (OpenAI). Use when the query is plain language."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Jarvis AI Search API

Complete API reference for 23blocks AI-powered natural language entity search using OpenAI ChatGPT.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://search.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### POST /jarvis/entities/search - AI Natural Language Search

Translates a natural language query into a structured entity search using OpenAI ChatGPT. The AI analyzes the user's intent, extracts search parameters, and executes the appropriate filtered search on the entity index.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/jarvis/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "articles about machine learning published this year by senior authors"
  }'
```

### Location-based Search
```bash
curl -X POST "$BLOCKS_API_URL/jarvis/entities/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"query_interpretation_error","title":"Could Not Interpret Query","detail":"The AI could not translate the query into structured search parameters."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-search`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// JarvisSearchService — client.search.jarvis
client.search.jarvis.search(query: JarvisSearchQuery): Promise<PageResult<JarvisSearchResult>>;
client.search.jarvis.suggest(query: string, limit?: number): Promise<string[]>;
client.search.jarvis.getRelated(entityUniqueId: string, entityType: string, limit?: number): Promise<JarvisSearchResult[]>;
```

### TypeScript Types

```typescript
import type {
  JarvisSearchQuery,
  JarvisSearchResult,
} from '@23blocks/block-search';
```

### React Hook

```typescript
import { useSearchBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useSearchBlock();
  const result = await client.search.jarvis.search({ query: 'find electronics under $50' });
}
```
