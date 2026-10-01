---
name: 23blocks-assets-warehouses-api
description: "Assets Block warehouses (storage locations): CRUD. Use when tracking where assets are kept."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Warehouses API

Complete API reference for 23blocks Assets Block warehouse management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://assets.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /warehouses/ - List Warehouses

Lists all warehouses.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/warehouses" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
        "description": "Primary storage facility",
        "address": "100 Industrial Blvd, Austin, TX",
        "city": "Austin",
        "state": "TX",
        "country": "US",
        "zip_code": "73301",
        "capacity": 500,
        "assets_count": 234,
        "contact_name": "Mike Johnson",
        "contact_email": "mike@warehouse.example.com",
        "contact_phone": "+1-555-0300",
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
curl -X GET "$BLOCKS_API_URL/warehouses/warehouse-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
      "description": "Primary storage facility",
      "address": "100 Industrial Blvd, Austin, TX",
      "city": "Austin",
      "state": "TX",
      "country": "US",
      "zip_code": "73301",
      "capacity": 500,
      "assets_count": 234,
      "contact_name": "Mike Johnson",
      "contact_email": "mike@warehouse.example.com",
      "contact_phone": "+1-555-0300",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
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
curl -X POST "$BLOCKS_API_URL/warehouses" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "warehouse": {
      "code": "ECD",
      "name": "East Coast Depot",
      "description": "Secondary storage for east coast operations",
      "address": "200 Logistics Dr, Newark, NJ",
      "city": "Newark",
      "state": "NJ",
      "country": "US",
      "zip_code": "07102",
      "capacity": 300,
      "contact_name": "Sarah Lee",
      "contact_email": "sarah@warehouse.example.com",
      "contact_phone": "+1-555-0400"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Warehouse code (unique identifier) |
| `name` | string | Yes | Warehouse name |
| `description` | string | No | Warehouse description |
| `address` | string | No | Street address |
| `city` | string | No | City |
| `state` | string | No | State or region |
| `country` | string | No | Country code |
| `zip_code` | string | No | Zip or postal code |
| `capacity` | integer | No | Maximum asset capacity |
| `contact_name` | string | No | Contact person |
| `contact_email` | string | No | Contact email |
| `contact_phone` | string | No | Contact phone |

**Response 201:**
```json
{
  "data": {
    "id": "new-warehouse-uuid",
    "type": "warehouse",
    "attributes": {
      "unique_id": "new-warehouse-uuid",
      "name": "East Coast Depot",
      "description": "Secondary storage for east coast operations",
      "address": "200 Logistics Dr, Newark, NJ",
      "city": "Newark",
      "state": "NJ",
      "country": "US",
      "assets_count": 0,
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
      "name": "Main Warehouse - Expanded",
      "capacity": 750
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "warehouse-uuid-123",
    "type": "warehouse",
    "attributes": {
      "unique_id": "warehouse-uuid-123",
      "name": "Main Warehouse - Expanded",
      "capacity": 750,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Warehouse not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /warehouses/:unique_id - Delete Warehouse

Deletes a warehouse.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/warehouses/warehouse-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** Returns `{}` with status 204

**Errors:**
- `404 Not Found` - Warehouse not found

---

## Data Models

### Warehouse
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `code` | string | Warehouse code (unique identifier) |
| `name` | string | Warehouse name |
| `description` | string | Warehouse description |
| `address` | string | Street address |
| `city` | string | City |
| `state` | string | State or region |
| `country` | string | Country code |
| `zip_code` | string | Zip or postal code |
| `capacity` | integer | Maximum asset capacity |
| `assets_count` | integer | Current number of assets stored |
| `contact_name` | string | Contact person name |
| `contact_email` | string | Contact email |
| `contact_phone` | string | Contact phone |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Error","detail":"Name can't be blank."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-assets`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useAssetsBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// WarehousesService — client.assets.warehouses
client.assets.warehouses.list(params?: ListWarehousesParams): Promise<PageResult<Warehouse>>;
client.assets.warehouses.get(uniqueId: string): Promise<Warehouse>;
client.assets.warehouses.create(data: CreateWarehouseRequest): Promise<Warehouse>;
client.assets.warehouses.update(uniqueId: string, data: UpdateWarehouseRequest): Promise<Warehouse>;
client.assets.warehouses.delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Warehouse,
  CreateWarehouseRequest,
  UpdateWarehouseRequest,
  ListWarehousesParams,
} from '@23blocks/block-assets';
```
