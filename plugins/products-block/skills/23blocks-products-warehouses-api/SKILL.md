---
name: 23blocks-products-warehouses-api
description: "Products Block warehouses: CRUD. Use for inventory locations."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Warehouses API

Complete API reference for 23blocks warehouse management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://products.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /warehouses/ - List Warehouses

Lists all warehouses.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/warehouses/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "warehouse-uuid-123",
      "type": "Warehouse",
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "warehouse-uuid-123",
    "type": "Warehouse",
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
    "type": "Warehouse",
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Warehouse Not Found","detail":"The requested warehouse could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-products`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
