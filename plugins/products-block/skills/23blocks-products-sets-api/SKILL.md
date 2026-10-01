---
name: 23blocks-products-sets-api
description: "Products Block product sets: products and categories in a set, recover. Use when grouping products into sets."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Sets API

Complete API reference for 23blocks product set management with product and category associations.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://products.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /sets/ - List Sets

Lists all product sets.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/sets/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "set-uuid-123",
      "type": "Set",
      "attributes": {
        "unique_id": "set-uuid-123",
        "name": "Office Starter Kit",
        "description": "Everything you need for a home office",
        "status": "active",
        "products_count": 5,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /sets/:unique_id/ - Get Set

Retrieves a single set by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/sets/set-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "set-uuid-123",
    "type": "Set",
    "attributes": {
      "unique_id": "set-uuid-123",
      "name": "Office Starter Kit",
      "description": "Everything you need for a home office",
      "status": "active",
      "products_count": 5,
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "products": {
        "data": [
          { "id": "product-uuid-001", "type": "Product" },
          { "id": "product-uuid-002", "type": "Product" }
        ]
      },
      "categories": {
        "data": [
          { "id": "category-uuid-001", "type": "Category" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Set not found

---

### POST /sets/ - Create Set

Creates a new product set.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/sets/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "set": {
      "name": "Gaming Bundle",
      "description": "Complete gaming setup"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Set name |
| `description` | string | No | Set description |

**Response 201:**
```json
{
  "data": {
    "id": "new-set-uuid",
    "type": "Set",
    "attributes": {
      "unique_id": "new-set-uuid",
      "name": "Gaming Bundle",
      "description": "Complete gaming setup",
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

### PUT /sets/:unique_id - Update Set

Updates an existing set.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/sets/set-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "set": {
      "name": "Premium Office Kit",
      "description": "Premium office setup bundle"
    }
  }'
```

**Response 200:** Updated set object

---

### DELETE /sets/:unique_id - Delete Set

Soft-deletes a set.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/sets/set-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /sets/:unique_id/recover - Recover Set

Recovers a soft-deleted set.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/sets/set-uuid-123/recover" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "set-uuid-123",
    "type": "Set",
    "attributes": {
      "unique_id": "set-uuid-123",
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Set Products

### POST /sets/:unique_id/products - Add Product to Set

Adds a product to a set.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/sets/set-uuid-123/products" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "product_unique_id": "product-uuid-001"
  }'
```

**Response 200:**
```json
{
  "message": "Product added to set successfully"
}
```

---

### DELETE /sets/:unique_id/products/:product_unique_id - Remove Product from Set

Removes a product from a set.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/sets/set-uuid-123/products/product-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Product removed from set successfully"
}
```

---

## Set Categories

### POST /sets/:unique_id/categories - Assign Category to Set

Assigns a category to a set.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/sets/set-uuid-123/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category_unique_id": "category-uuid-001"
  }'
```

**Response 200:**
```json
{
  "message": "Category assigned to set successfully"
}
```

---

### DELETE /sets/:unique_id/categories/:category_unique_id - Remove Category from Set

Removes a category from a set.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/sets/set-uuid-123/categories/category-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Category removed from set successfully"
}
```

---

## Data Models

### Set
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Set name |
| `description` | string | Set description |
| `status` | enum | active, inactive, trash |
| `products_count` | integer | Number of products in set |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Set Not Found","detail":"The requested set could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-products`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useProductsBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// ProductSetsService — client.products.productSets
list(params?: ListProductSetsParams): Promise<PageResult<ProductSet>>;
get(uniqueId: string): Promise<ProductSet>;
create(data: CreateProductSetRequest): Promise<ProductSet>;
update(uniqueId: string, data: UpdateProductSetRequest): Promise<ProductSet>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<ProductSet>;
addProduct(uniqueId: string, productUniqueId: string, quantity?: number): Promise<ProductSet>;
removeProduct(uniqueId: string, productUniqueId: string): Promise<void>;
addCategory(uniqueId: string, categoryUniqueId: string): Promise<ProductSet>;
removeCategory(uniqueId: string, categoryUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  ProductSet,
  CreateProductSetRequest,
  UpdateProductSetRequest,
  ListProductSetsParams,
} from '@23blocks/block-products';
```
