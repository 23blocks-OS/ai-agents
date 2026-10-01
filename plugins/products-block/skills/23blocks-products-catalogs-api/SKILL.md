---
name: 23blocks-products-catalogs-api
description: "Products Block catalogs: CRUD. Use when grouping products into catalogs."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Catalogs API

Complete API reference for 23blocks product catalog management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://products.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /catalogs/ - List Catalogs

Lists all product catalogs.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/catalogs/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "catalog-uuid-123",
      "type": "Catalog",
      "attributes": {
        "unique_id": "catalog-uuid-123",
        "name": "Summer Collection 2025",
        "description": "Products for the summer season",
        "status": "active",
        "products_count": 85,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /catalogs/:unique_id/ - Get Catalog

Retrieves a single catalog by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/catalogs/catalog-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "catalog-uuid-123",
    "type": "Catalog",
    "attributes": {
      "unique_id": "catalog-uuid-123",
      "name": "Summer Collection 2025",
      "description": "Products for the summer season",
      "status": "active",
      "products_count": 85,
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
- `404 Not Found` - Catalog not found

---

### POST /catalogs/ - Create Catalog

Creates a new product catalog.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/catalogs/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "catalog": {
      "name": "Holiday Collection 2025",
      "description": "Products for the holiday season"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Catalog name |
| `description` | string | No | Catalog description |

**Response 201:**
```json
{
  "data": {
    "id": "new-catalog-uuid",
    "type": "Catalog",
    "attributes": {
      "unique_id": "new-catalog-uuid",
      "name": "Holiday Collection 2025",
      "description": "Products for the holiday season",
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

### PUT /catalogs/:unique_id - Update Catalog

Updates an existing catalog.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/catalogs/catalog-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "catalog": {
      "name": "Updated Catalog Name",
      "description": "Updated description"
    }
  }'
```

**Response 200:** Updated catalog object

---

### DELETE /catalogs/:unique_id - Delete Catalog

Deletes a catalog.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/catalogs/catalog-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Catalog
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Catalog name |
| `description` | string | Catalog description |
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
// ChannelsService — client.products.channels
list(): Promise<Channel[]>;
get(uniqueId: string): Promise<Channel>;
```

### TypeScript Types

```typescript
import type {
  Channel,
  ProductCatalog,
} from '@23blocks/block-products';
```
