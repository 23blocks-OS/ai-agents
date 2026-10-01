---
name: 23blocks-products-promotions-api
description: "Products Block promotions on products. Use for discounts and special offers."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Promotions API

Complete API reference for 23blocks product promotion management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://products.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /products/:unique_id/promotions - List Promotions

Lists all promotions for a product.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/promotions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "promo-uuid-123",
      "type": "Promotion",
      "attributes": {
        "unique_id": "promo-uuid-123",
        "product_unique_id": "product-uuid-123",
        "name": "Summer Sale",
        "description": "20% off for summer",
        "discount_type": "percentage",
        "discount_value": 20.0,
        "starts_at": "2025-06-01T00:00:00Z",
        "ends_at": "2025-08-31T23:59:59Z",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /products/:unique_id/promotions/:promotion_unique_id - Get Promotion

Retrieves a specific promotion.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/promotions/promo-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "promo-uuid-123",
    "type": "Promotion",
    "attributes": {
      "unique_id": "promo-uuid-123",
      "product_unique_id": "product-uuid-123",
      "name": "Summer Sale",
      "description": "20% off for summer",
      "discount_type": "percentage",
      "discount_value": 20.0,
      "min_quantity": 1,
      "max_uses": 100,
      "current_uses": 42,
      "starts_at": "2025-06-01T00:00:00Z",
      "ends_at": "2025-08-31T23:59:59Z",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Promotion not found

---

### POST /products/:unique_id/promotions - Create Promotion

Creates a new promotion on a product.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/promotions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "promotion": {
      "name": "Holiday Special",
      "description": "15% off for holidays",
      "discount_type": "percentage",
      "discount_value": 15.0,
      "min_quantity": 1,
      "max_uses": 200,
      "starts_at": "2025-12-01T00:00:00Z",
      "ends_at": "2025-12-31T23:59:59Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Promotion name |
| `description` | string | No | Promotion description |
| `discount_type` | enum | Yes | percentage, fixed_amount |
| `discount_value` | decimal | Yes | Discount value |
| `min_quantity` | integer | No | Minimum quantity required |
| `max_uses` | integer | No | Maximum number of uses |
| `starts_at` | timestamp | No | Promotion start date |
| `ends_at` | timestamp | No | Promotion end date |

**Response 201:**
```json
{
  "data": {
    "id": "new-promo-uuid",
    "type": "Promotion",
    "attributes": {
      "unique_id": "new-promo-uuid",
      "product_unique_id": "product-uuid-123",
      "name": "Holiday Special",
      "discount_type": "percentage",
      "discount_value": 15.0,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /products/:unique_id/promotions/:promotion_unique_id - Update Promotion

Updates an existing promotion.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/promotions/promo-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "promotion": {
      "discount_value": 25.0,
      "ends_at": "2025-09-15T23:59:59Z"
    }
  }'
```

**Response 200:** Updated promotion object

---

### DELETE /products/:unique_id/promotions/:promotion_unique_id - Delete Promotion

Deletes a promotion.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/products/product-uuid-123/promotions/promo-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Promotion
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `product_unique_id` | uuid | Product identifier |
| `name` | string | Promotion name |
| `description` | string | Promotion description |
| `discount_type` | enum | percentage, fixed_amount |
| `discount_value` | decimal | Discount value |
| `min_quantity` | integer | Minimum quantity required |
| `max_uses` | integer | Maximum uses allowed |
| `current_uses` | integer | Current number of uses |
| `starts_at` | timestamp | Start date |
| `ends_at` | timestamp | End date |
| `status` | enum | active, inactive, expired |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Error","detail":"Discount value must be greater than zero."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-products`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// ProductPromotionsService — client.products.promotions
list(params?: ListProductPromotionsParams): Promise<PageResult<ProductPromotion>>;
get(uniqueId: string): Promise<ProductPromotion>;
create(data: CreateProductPromotionRequest): Promise<ProductPromotion>;
update(uniqueId: string, data: UpdateProductPromotionRequest): Promise<ProductPromotion>;
delete(uniqueId: string): Promise<void>;
activate(uniqueId: string): Promise<ProductPromotion>;
deactivate(uniqueId: string): Promise<ProductPromotion>;
```

### TypeScript Types

```typescript
import type {
  ProductPromotion,
  CreateProductPromotionRequest,
  UpdateProductPromotionRequest,
  ListProductPromotionsParams,
} from '@23blocks/block-products';
```

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: list active promotions
  const result = await client.products.promotions.list({ active: true });
}
```
