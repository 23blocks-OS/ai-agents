---
name: 23blocks-search-api
description: "Search Block full-text search on the MasterIndex and recent queries. Use for keyword search."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Search API

Complete API reference for 23blocks full-text search on the MasterIndex.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://search.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### POST /search - Execute Search Query

Executes a full-text search query on the MasterIndex. The search supports narrow (AND) and wide (OR) modes and automatically logs the query with performance metrics.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Invalid Query","detail":"Search query cannot be empty."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-search`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// SearchService — client.search.search
client.search.search.search(request: SearchRequest): Promise<SearchResponse>;
client.search.search.suggest(query: string, limit?: number): Promise<SearchResult[]>;
client.search.search.entityTypes(): Promise<EntityType[]>;

// SearchHistoryService — client.search.history
client.search.history.recent(limit?: number): Promise<LastQuery[]>;
client.search.history.get(uniqueId: string): Promise<SearchQuery>;
client.search.history.clear(): Promise<void>;
client.search.history.delete(uniqueId: string): Promise<void>;

// FavoritesService — client.search.favorites
client.search.favorites.list(params?: ListParams): Promise<PageResult<FavoriteEntity>>;
client.search.favorites.get(uniqueId: string): Promise<FavoriteEntity>;
client.search.favorites.add(request: AddFavoriteRequest): Promise<FavoriteEntity>;
client.search.favorites.remove(uniqueId: string): Promise<void>;
client.search.favorites.isFavorite(entityUniqueId: string): Promise<boolean>;
```

### TypeScript Types

```typescript
import type {
  SearchResult,
  SearchQuery,
  LastQuery,
  FavoriteEntity,
  EntityType,
  SearchRequest,
  SearchResponse,
  AddFavoriteRequest,
} from '@23blocks/block-search';
```

### React Hook

```typescript
import { useSearchBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useSearchBlock();
  const result = await client.search.search.search({ query: 'hello world' });
}
```
