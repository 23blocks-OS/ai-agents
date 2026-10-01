---
name: 23blocks-products-vendors-api
description: "Products Block vendors: CRUD, load or update a vendor's product catalog. Use for supplier catalogs."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Vendors API

Complete API reference for 23blocks product vendor management with product catalog operations.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://products.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /vendors/ - List Vendors

Lists all vendors.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/vendors/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "vendor-uuid-123",
      "type": "Vendor",
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "vendor-uuid-123",
    "type": "Vendor",
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
    "type": "Vendor",
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
    "type": "BatchResult",
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
    "type": "BatchResult",
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Vendor Not Found","detail":"The requested vendor could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-products`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
