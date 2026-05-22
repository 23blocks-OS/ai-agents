---
name: 23blocks-products-promotions-api
description: Manage 23blocks product promotions via REST API. Use when creating, updating, deleting, or listing promotions on products for discounts and special offers.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Promotions API

Complete API reference for 23blocks product promotion management.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Products API base URL | `https://products.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

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

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Discount value must be greater than zero."
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
