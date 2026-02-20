---
name: products-warehouses-api
description: Manage 23blocks warehouses via REST API. Use when creating, updating, deleting, or listing warehouses for inventory management.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Warehouses API

Complete API reference for 23blocks warehouse management.

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
curl -X GET "$BLOCKS_API_URL/warehouses/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /warehouses/ - List Warehouses

Lists all warehouses.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/warehouses/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "warehouse-uuid-123",
      "type": "warehouse",
      "attributes": {
        "unique_id": "warehouse-uuid-123",
        "name": "Main Warehouse",
        "code": "WH-MAIN",
        "address": "100 Distribution Rd",
        "city": "Miami",
        "state": "FL",
        "country": "US",
        "zip_code": "33101",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /warehouses/:unique_id/ - Get Warehouse

Retrieves a single warehouse by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/warehouses/warehouse-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "warehouse-uuid-123",
    "type": "warehouse",
    "attributes": {
      "unique_id": "warehouse-uuid-123",
      "name": "Main Warehouse",
      "code": "WH-MAIN",
      "address": "100 Distribution Rd",
      "city": "Miami",
      "state": "FL",
      "country": "US",
      "zip_code": "33101",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Warehouse not found

---

### POST /warehouses/ - Create Warehouse

Creates a new warehouse.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/warehouses/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "warehouse": {
      "name": "West Coast Warehouse",
      "code": "WH-WEST",
      "address": "200 Logistics Blvd",
      "city": "Los Angeles",
      "state": "CA",
      "country": "US",
      "zip_code": "90001"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Warehouse name |
| `code` | string | No | Warehouse code |
| `address` | string | No | Street address |
| `city` | string | No | City |
| `state` | string | No | State/province |
| `country` | string | No | Country code |
| `zip_code` | string | No | Postal code |

**Response 201:**
```json
{
  "data": {
    "id": "new-warehouse-uuid",
    "type": "warehouse",
    "attributes": {
      "unique_id": "new-warehouse-uuid",
      "name": "West Coast Warehouse",
      "code": "WH-WEST",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /warehouses/:unique_id - Update Warehouse

Updates an existing warehouse.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/warehouses/warehouse-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "warehouse": {
      "name": "Main Distribution Center",
      "address": "101 Distribution Rd"
    }
  }'
```

**Response 200:** Updated warehouse object

---

### DELETE /warehouses/:unique_id - Delete Warehouse

Deletes a warehouse.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/warehouses/warehouse-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Warehouse
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Warehouse name |
| `code` | string | Warehouse code |
| `address` | string | Street address |
| `city` | string | City |
| `state` | string | State/province |
| `country` | string | Country code |
| `zip_code` | string | Postal code |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Warehouse Not Found",
    "detail": "The requested warehouse could not be found."
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
// WarehousesService — client.products.warehouses
list(params?: ListWarehousesParams): Promise<PageResult<Warehouse>>;
get(uniqueId: string): Promise<Warehouse>;
create(data: CreateWarehouseRequest): Promise<Warehouse>;
update(uniqueId: string, data: UpdateWarehouseRequest): Promise<Warehouse>;
delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Warehouse,
  CreateWarehouseRequest,
  UpdateWarehouseRequest,
  ListWarehousesParams,
} from '@23blocks/block-products';
```

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: list warehouses for a vendor
  const result = await client.products.warehouses.list({ vendorUniqueId: 'vendor-uuid' });
}
```
