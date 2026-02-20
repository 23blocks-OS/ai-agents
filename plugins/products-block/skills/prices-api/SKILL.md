---
name: products-prices-api
description: Manage 23blocks product pricing via REST API. Use when creating or updating product prices, managing variation prices, handling product variations, or managing pricing channels.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Prices API

Complete API reference for 23blocks product pricing management with variations and channels.

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
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/prices" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Product Price Endpoints

### GET /products/:product_unique_id/prices - List Prices

Lists all prices for a product.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/prices" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "price-uuid-123",
      "type": "price",
      "attributes": {
        "unique_id": "price-uuid-123",
        "product_unique_id": "product-uuid-123",
        "amount": 49.99,
        "compare_at_amount": 69.99,
        "currency": "USD",
        "channel_unique_id": "channel-uuid-001",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /products/:product_unique_id/prices/:price_unique_id - Get Price

Retrieves a specific price entry.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/prices/price-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "price-uuid-123",
    "type": "price",
    "attributes": {
      "unique_id": "price-uuid-123",
      "product_unique_id": "product-uuid-123",
      "amount": 49.99,
      "compare_at_amount": 69.99,
      "currency": "USD",
      "channel_unique_id": "channel-uuid-001",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Price not found

---

### POST /products/:product_unique_id/prices/ - Create Price

Creates a new price for a product.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/prices/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "price": {
      "amount": 39.99,
      "compare_at_amount": 59.99,
      "currency": "USD",
      "channel_unique_id": "channel-uuid-001"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount` | decimal | Yes | Price amount |
| `compare_at_amount` | decimal | No | Original/compare price |
| `currency` | string | Yes | Currency code (USD, EUR, etc.) |
| `channel_unique_id` | uuid | No | Pricing channel |

**Response 201:**
```json
{
  "data": {
    "id": "new-price-uuid",
    "type": "price",
    "attributes": {
      "unique_id": "new-price-uuid",
      "amount": 39.99,
      "compare_at_amount": 59.99,
      "currency": "USD",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /products/:product_unique_id/prices/:price_unique_id - Update Price

Updates an existing price.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/prices/price-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "price": {
      "amount": 44.99,
      "compare_at_amount": 64.99
    }
  }'
```

**Response 200:** Updated price object

---

## Variation Price Endpoints

### GET /products/:unique_id/variations/:variation_unique_id/prices - List Variation Prices

Lists all prices for a product variation.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/variations/variation-uuid-001/prices" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "vprice-uuid-123",
      "type": "variation_price",
      "attributes": {
        "unique_id": "vprice-uuid-123",
        "variation_unique_id": "variation-uuid-001",
        "amount": 54.99,
        "currency": "USD",
        "channel_unique_id": "channel-uuid-001",
        "status": "active"
      }
    }
  ]
}
```

---

### POST /products/:unique_id/variations/:variation_unique_id/prices/ - Create Variation Price

Creates a price for a product variation.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/variations/variation-uuid-001/prices/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "price": {
      "amount": 54.99,
      "currency": "USD",
      "channel_unique_id": "channel-uuid-001"
    }
  }'
```

**Response 201:** Created variation price object

---

### PUT /products/:unique_id/variations/:variation_unique_id/prices/:price_unique_id - Update Variation Price

Updates a variation price.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/variations/variation-uuid-001/prices/vprice-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "price": {
      "amount": 59.99
    }
  }'
```

**Response 200:** Updated variation price object

---

## Variation Endpoints

### GET /products/:unique_id/variations - List Variations

Lists all variations for a product.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/variations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "variation-uuid-001",
      "type": "variation",
      "attributes": {
        "unique_id": "variation-uuid-001",
        "product_unique_id": "product-uuid-123",
        "name": "Large - Black",
        "sku": "KB-001-LG-BLK",
        "barcode": "1234567890124",
        "options": {"size": "large", "color": "black"},
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /products/:unique_id/variations - Create Variation

Creates a new product variation.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/variations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "variation": {
      "name": "Small - White",
      "sku": "KB-001-SM-WHT",
      "barcode": "1234567890125",
      "options": {"size": "small", "color": "white"}
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Variation name |
| `sku` | string | No | Variation SKU |
| `barcode` | string | No | Variation barcode |
| `options` | object | No | Key-value option pairs |

**Response 201:**
```json
{
  "data": {
    "id": "new-variation-uuid",
    "type": "variation",
    "attributes": {
      "unique_id": "new-variation-uuid",
      "name": "Small - White",
      "sku": "KB-001-SM-WHT",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /products/:unique_id/variations/:variation_unique_id - Update Variation

Updates a product variation.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/variations/variation-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "variation": {
      "name": "Large - Midnight Black",
      "sku": "KB-001-LG-MBLK"
    }
  }'
```

**Response 200:** Updated variation object

---

### DELETE /products/:unique_id/variations/:variation_unique_id - Delete Variation

Deletes a product variation.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/products/product-uuid-123/variations/variation-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Channel Endpoints

### GET /channels/ - List Channels

Lists all pricing channels.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/channels/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "channel-uuid-001",
      "type": "channel",
      "attributes": {
        "unique_id": "channel-uuid-001",
        "name": "Online Store",
        "code": "ONLINE",
        "currency": "USD",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /channels/ - Create Channel

Creates a new pricing channel.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/channels/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": {
      "name": "Wholesale",
      "code": "WHOLESALE",
      "currency": "USD"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Channel name |
| `code` | string | No | Channel code |
| `currency` | string | No | Default currency |

**Response 201:** Created channel object

---

### PUT /channels/:unique_id - Update Channel

Updates a pricing channel.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/channels/channel-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": {
      "name": "Updated Channel Name"
    }
  }'
```

**Response 200:** Updated channel object

---

### DELETE /channels/:unique_id - Delete Channel

Deletes a pricing channel.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/channels/channel-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Price
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `product_unique_id` | uuid | Product identifier |
| `amount` | decimal | Price amount |
| `compare_at_amount` | decimal | Original/compare price |
| `currency` | string | Currency code |
| `channel_unique_id` | uuid | Pricing channel |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Variation
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `product_unique_id` | uuid | Parent product |
| `name` | string | Variation name |
| `sku` | string | Variation SKU |
| `barcode` | string | Variation barcode |
| `options` | object | Key-value option pairs |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |

### Channel
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Channel name |
| `code` | string | Channel code |
| `currency` | string | Default currency |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Amount must be greater than zero."
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
// ProductPricesService — client.products.prices
list(params?: ListProductPricesParams): Promise<PageResult<ProductPrice>>;
get(uniqueId: string): Promise<ProductPrice>;
create(data: CreateProductPriceRequest): Promise<ProductPrice>;
update(uniqueId: string, data: UpdateProductPriceRequest): Promise<ProductPrice>;
delete(uniqueId: string): Promise<void>;
getForProduct(productUniqueId: string, params?: ListProductPricesParams): Promise<PageResult<ProductPrice>>;
getForVariation(variationUniqueId: string, params?: ListProductPricesParams): Promise<PageResult<ProductPrice>>;

// ProductVariationsService — client.products.variations
list(productUniqueId: string): Promise<ProductVariation[]>;
get(productUniqueId: string, variationUniqueId: string): Promise<ProductVariation>;
create(productUniqueId: string, data: CreateVariationRequest): Promise<ProductVariation>;
update(productUniqueId: string, variationUniqueId: string, data: UpdateVariationRequest): Promise<ProductVariation>;
delete(productUniqueId: string, variationUniqueId: string): Promise<void>;
listReviews(productUniqueId: string, variationUniqueId: string): Promise<PageResult<any>>;
createReview(productUniqueId: string, variationUniqueId: string, data: { rating: number; title?: string; content?: string }): Promise<any>;
updateReview(productUniqueId: string, variationUniqueId: string, reviewUniqueId: string, data: { rating?: number; title?: string; content?: string }): Promise<any>;
deleteReview(productUniqueId: string, variationUniqueId: string, reviewUniqueId: string): Promise<void>;
flagReview(productUniqueId: string, variationUniqueId: string, reviewUniqueId: string): Promise<any>;

// ChannelsService — client.products.channels
list(): Promise<Channel[]>;
get(uniqueId: string): Promise<Channel>;
```

### TypeScript Types

```typescript
import type {
  ProductPrice,
  CreateProductPriceRequest,
  UpdateProductPriceRequest,
  ListProductPricesParams,
  ProductVariation,
  CreateVariationRequest,
  UpdateVariationRequest,
  Channel,
} from '@23blocks/block-products';
```

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: get prices for a product
  const result = await client.products.prices.getForProduct('product-uuid-123');
}
```
