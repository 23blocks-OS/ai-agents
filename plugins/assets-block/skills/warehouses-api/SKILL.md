---
name: assets-warehouses-api
description: Create and manage 23blocks asset warehouses via REST API. Use when creating, listing, updating, or deleting warehouses (storage locations) for asset tracking.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Warehouses API

Complete API reference for 23blocks Assets Block warehouse management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://assets.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Assets API base URL | `https://assets.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/warehouses" \
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
curl -X GET "$BLOCKS_API_URL/warehouses" \
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
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "warehouse": {
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
  -H "AppId: $BLOCKS_API_KEY" \
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
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Warehouse not found

---

## Data Models

### Warehouse
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
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
npm install @23blocks/block-assets
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

### React Hook

```typescript
import { useAssetsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAssetsBlock();
  const result = await client.assets.warehouses.list();
}
```
