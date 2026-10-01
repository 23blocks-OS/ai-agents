---
name: 23blocks-products-collections-api
description: "Products Block collections: CRUD. Use for curated product collections."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Collections API

Complete API reference for 23blocks product collection management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://products.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /collections/ - List Collections

Lists all product collections.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/collections/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "collection-uuid-123",
      "type": "Collection",
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "collection-uuid-123",
    "type": "Collection",
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
          { "id": "product-uuid-001", "type": "Product" }
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
    "type": "Collection",
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Error","detail":"Name can't be blank."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-products`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useProductsBlock()` from `@23blocks/react`.

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
