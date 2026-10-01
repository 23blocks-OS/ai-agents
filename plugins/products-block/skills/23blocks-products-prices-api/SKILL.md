---
name: 23blocks-products-prices-api
description: "Products Block prices: product and variation prices, variations, channels. Use when setting prices."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Prices API

Complete API reference for 23blocks product pricing management with variations and channels.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://products.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/products/:product_unique_id/prices` | List all prices for a product |
| GET | `/products/:product_unique_id/prices/:price_unique_id` | Get a specific price |
| POST | `/products/:product_unique_id/prices/` | Create a new product price |
| PUT | `/products/:product_unique_id/prices/:price_unique_id` | Update an existing price |
| GET | `/products/:unique_id/variations/:variation_unique_id/prices` | List variation prices |
| POST | `/products/:unique_id/variations/:variation_unique_id/prices/` | Create variation price |
| PUT | `/products/:unique_id/variations/:variation_unique_id/prices/:price_unique_id` | Update variation price |
| GET | `/products/:unique_id/variations` | List product variations |
| POST | `/products/:unique_id/variations` | Create a variation |
| PUT | `/products/:unique_id/variations/:variation_unique_id` | Update a variation |
| DELETE | `/products/:unique_id/variations/:variation_unique_id` | Delete a variation |
| GET | `/channels/` | List pricing channels |
| POST | `/channels/` | Create a pricing channel |
| PUT | `/channels/:unique_id` | Update a pricing channel |
| DELETE | `/channels/:unique_id` | Delete a pricing channel |

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Error","detail":"Amount must be greater than zero."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-products`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
