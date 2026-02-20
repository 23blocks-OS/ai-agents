---
name: products-collections-api
description: Manage 23blocks product collections via REST API. Use when creating, updating, deleting, or listing product collections.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Collections API

Complete API reference for 23blocks product collection management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://products.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Products API base URL | `https://products.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/collections/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /collections/ - List Collections

Lists all product collections.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/collections/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "collection-uuid-123",
      "type": "collection",
      "attributes": {
        "unique_id": "collection-uuid-123",
        "name": "Best Sellers",
        "description": "Our top selling products",
        "status": "active",
        "products_count": 25,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /collections/:unique_id/ - Get Collection

Retrieves a single collection by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/collections/collection-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "collection-uuid-123",
    "type": "collection",
    "attributes": {
      "unique_id": "collection-uuid-123",
      "name": "Best Sellers",
      "description": "Our top selling products",
      "status": "active",
      "products_count": 25,
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "products": {
        "data": [
          { "id": "product-uuid-001", "type": "product" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Collection not found

---

### POST /collections/ - Create Collection

Creates a new product collection.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/collections/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "collection": {
      "name": "New Arrivals",
      "description": "Recently added products"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Collection name |
| `description` | string | No | Collection description |

**Response 201:**
```json
{
  "data": {
    "id": "new-collection-uuid",
    "type": "collection",
    "attributes": {
      "unique_id": "new-collection-uuid",
      "name": "New Arrivals",
      "description": "Recently added products",
      "status": "active",
      "products_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /collections/:unique_id - Update Collection

Updates an existing collection.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/collections/collection-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "collection": {
      "name": "Updated Collection Name",
      "description": "Updated description"
    }
  }'
```

**Response 200:** Updated collection object

---

### DELETE /collections/:unique_id - Delete Collection

Deletes a collection.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/collections/collection-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Collection
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Collection name |
| `description` | string | Collection description |
| `status` | enum | active, inactive |
| `products_count` | integer | Number of products |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Name can't be blank."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-products
```

### Setup

```typescript
import { create23BlocksClient } from '@23blocks/sdk';

const client = create23BlocksClient({
  authToken: process.env.BLOCKS_AUTH_TOKEN!,
  apiKey: process.env.BLOCKS_API_KEY!,
  apiUrl: process.env.BLOCKS_API_URL!,
});
```

### Available Methods

```typescript
// CollectionsService — client.products.collections
list(): Promise<Collection[]>;
get(uniqueId: string): Promise<Collection>;
```

### TypeScript Types

```typescript
import type {
  Collection,
} from '@23blocks/block-products';
```

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: list all collections
  const result = await client.products.collections.list();
}
```
