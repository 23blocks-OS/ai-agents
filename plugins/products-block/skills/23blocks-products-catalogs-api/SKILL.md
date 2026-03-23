---
name: 23blocks-products-catalogs-api
description: Manage 23blocks product catalogs via REST API. Use when creating, updating, deleting, or listing product catalogs for grouping products.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Catalogs API

Complete API reference for 23blocks product catalog management.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Products API base URL | `https://products.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://products.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://products.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### GET /catalogs/ - List Catalogs

Lists all product catalogs.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/catalogs/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "catalog-uuid-123",
      "type": "catalog",
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
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "catalog-uuid-123",
    "type": "catalog",
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
          { "id": "product-uuid-001", "type": "product" }
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
  -H "AppId: $BLOCKS_API_KEY" \
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
    "type": "catalog",
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
  -H "AppId: $BLOCKS_API_KEY" \
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
  -H "AppId: $BLOCKS_API_KEY"
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

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: list all channels
  const result = await client.products.channels.list();
}
```
