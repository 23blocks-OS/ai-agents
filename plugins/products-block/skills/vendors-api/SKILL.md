---
name: products-vendors-api
description: Manage 23blocks product vendors via REST API. Use when creating, updating, deleting vendors, or loading and updating vendor product catalogs.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Vendors API

Complete API reference for 23blocks product vendor management with product catalog operations.

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
curl -X GET "$BLOCKS_API_URL/vendors/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /vendors/ - List Vendors

Lists all vendors.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/vendors/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "vendor-uuid-123",
      "type": "vendor",
      "attributes": {
        "unique_id": "vendor-uuid-123",
        "name": "Main Supplier Inc",
        "code": "MSI",
        "contact_email": "orders@supplier.com",
        "contact_phone": "+1234567890",
        "address": "123 Supplier St",
        "status": "active",
        "products_count": 250,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /vendors/:unique_id/ - Get Vendor

Retrieves a single vendor by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/vendors/vendor-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "vendor-uuid-123",
    "type": "vendor",
    "attributes": {
      "unique_id": "vendor-uuid-123",
      "name": "Main Supplier Inc",
      "code": "MSI",
      "contact_email": "orders@supplier.com",
      "contact_phone": "+1234567890",
      "address": "123 Supplier St",
      "status": "active",
      "products_count": 250,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Vendor not found

---

### POST /vendors/ - Create Vendor

Creates a new vendor.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/vendors/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "vendor": {
      "name": "New Supplier Co",
      "code": "NSC",
      "contact_email": "info@newsupplier.com",
      "contact_phone": "+1987654321",
      "address": "456 Vendor Ave"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Vendor name |
| `code` | string | No | Vendor code |
| `contact_email` | string | No | Contact email |
| `contact_phone` | string | No | Contact phone |
| `address` | string | No | Vendor address |

**Response 201:**
```json
{
  "data": {
    "id": "new-vendor-uuid",
    "type": "vendor",
    "attributes": {
      "unique_id": "new-vendor-uuid",
      "name": "New Supplier Co",
      "code": "NSC",
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

### PUT /vendors/:unique_id - Update Vendor

Updates an existing vendor.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/vendors/vendor-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "vendor": {
      "name": "Updated Supplier Inc",
      "contact_email": "new-orders@supplier.com"
    }
  }'
```

**Response 200:** Updated vendor object

---

### DELETE /vendors/:unique_id - Delete Vendor

Deletes a vendor.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/vendors/vendor-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Vendor Products

### POST /vendors/:unique_id/products/load - Load Products

Loads a batch of products into a vendor's catalog.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/vendors/vendor-uuid-123/products/load" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "products": [
      {
        "product_unique_id": "product-uuid-001",
        "vendor_sku": "V-KB-001",
        "cost": 25.00
      },
      {
        "product_unique_id": "product-uuid-002",
        "vendor_sku": "V-MS-001",
        "cost": 12.50
      }
    ]
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `products` | array | Yes | Array of product entries |
| `products[].product_unique_id` | uuid | Yes | Product identifier |
| `products[].vendor_sku` | string | No | Vendor-specific SKU |
| `products[].cost` | decimal | No | Vendor cost price |

**Response 200:**
```json
{
  "data": {
    "type": "batch_result",
    "attributes": {
      "loaded": 2,
      "errors": 0
    }
  }
}
```

---

### PUT /vendors/:unique_id/products/update - Update Products

Updates vendor product catalog entries.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/vendors/vendor-uuid-123/products/update" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "products": [
      {
        "product_unique_id": "product-uuid-001",
        "vendor_sku": "V-KB-002",
        "cost": 27.50
      }
    ]
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "batch_result",
    "attributes": {
      "updated": 1,
      "errors": 0
    }
  }
}
```

---

## Data Models

### Vendor
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Vendor name |
| `code` | string | Vendor code |
| `contact_email` | string | Contact email |
| `contact_phone` | string | Contact phone |
| `address` | string | Vendor address |
| `status` | enum | active, inactive |
| `products_count` | integer | Number of products |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Vendor Not Found",
    "detail": "The requested vendor could not be found."
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
// VendorsService — client.products.vendors
list(params?: ListVendorsParams): Promise<PageResult<Vendor>>;
get(uniqueId: string): Promise<Vendor>;
create(data: CreateVendorRequest): Promise<Vendor>;
update(uniqueId: string, data: UpdateVendorRequest): Promise<Vendor>;
delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Vendor,
  CreateVendorRequest,
  UpdateVendorRequest,
  ListVendorsParams,
} from '@23blocks/block-products';
```

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: list vendors with search
  const result = await client.products.vendors.list({ search: 'supplier' });
}
```
