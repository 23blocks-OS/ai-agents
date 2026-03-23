---
name: 23blocks-products-api
description: Manage 23blocks products via REST API. Use when creating, updating, searching products, managing product images, assigning categories and catalogs, handling suggestions, addons, replacements, and product filters.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Products API

Complete API reference for 23blocks product management with images, associations, and search.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Products API base URL | `https://products.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

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


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/products/` | List products with pagination and search |
| GET | `/products/:unique_id/` | Get a single product |
| POST | `/products/` | Create a new product |
| PUT | `/products/:unique_id` | Update an existing product |
| DELETE | `/products/:unique_id` | Soft-delete a product |
| PUT | `/products/:unique_id/recover` | Recover a soft-deleted product |
| GET | `/products/trash/show` | List soft-deleted products |
| GET | `/catalog/` | Catalog-optimized product view |
| POST | `/products/search` | Full-text search across products |
| GET | `/products/:unique_id/replacements` | List replacement products |
| GET | `/tools/products/payload/` | Get product payload structure |
| GET | `/tools/products/payload/filters` | Get available payload filter fields |
| GET | `/tools/products/filters` | List saved product filters |
| POST | `/tools/products/filters` | Create a product filter |
| PUT | `/tools/products/filters/:unique_id` | Update a product filter |
| DELETE | `/tools/products/filters/:unique_id` | Delete a product filter |
| PUT | `/products/:unique_id/presign` | Get presigned URL for image upload |
| POST | `/products/:unique_id/images` | Register an uploaded image |
| GET | `/products/:unique_id/images` | List product images |
| PUT | `/products/:unique_id/images/:unique_file_id` | Update image metadata |
| DELETE | `/products/:unique_id/images/:unique_file_id` | Delete a product image |
| PUT | `/products/:unique_id/images/:unique_file_id/approve` | Approve a product image |
| PUT | `/products/:unique_id/images/:unique_file_id/publish` | Publish a product image |
| POST | `/products/:unique_id/categories` | Assign category to product |
| DELETE | `/products/:unique_id/categories/:category_unique_id` | Remove category from product |
| POST | `/products/:unique_id/catalogs` | Assign product to catalog |
| GET | `/products/:unique_id/suggestions` | List suggested products |
| POST | `/products/:unique_id/suggestions` | Add a product suggestion |
| DELETE | `/products/:unique_id/suggestions/:product_unique_id` | Remove a product suggestion |
| GET | `/products/:unique_id/addons` | List product addons |
| POST | `/products/:unique_id/addons` | Add an addon to a product |
| DELETE | `/products/:unique_id/addons/:addon_unique_id` | Remove an addon |
| POST | `/products/:unique_id/replacements` | Add a replacement product |
| GET | `/products/:unique_id/vendors` | List vendors for a product |

---

## Data Models

### Product
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Product name |
| `description` | string | Product description |
| `sku` | string | Stock keeping unit |
| `barcode` | string | Product barcode |
| `brand_id` | uuid | Brand identifier |
| `category_id` | uuid | Category identifier |
| `status` | enum | active, inactive, trash |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### ProductImage
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `file_url` | string | Image URL |
| `alt_text` | string | Alternative text |
| `position` | integer | Display order |
| `status` | enum | pending, approved, published |
| `created_at` | timestamp | Creation time |

### ProductFilter
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Filter name |
| `criteria` | object | Filter criteria |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Product Not Found",
    "detail": "The requested product could not be found."
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
// ProductsService — client.products.products
list(params?: ListProductsParams): Promise<PageResult<Product>>;
get(uniqueId: string): Promise<Product>;
create(data: CreateProductRequest): Promise<Product>;
update(uniqueId: string, data: UpdateProductRequest): Promise<Product>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<Product>;
search(query: string, params?: ListProductsParams): Promise<PageResult<Product>>;
listDeleted(params?: ListProductsParams): Promise<PageResult<Product>>;
listVariations(productUniqueId: string): Promise<ProductVariation[]>;
getVariation(uniqueId: string): Promise<ProductVariation>;
createVariation(data: CreateVariationRequest): Promise<ProductVariation>;
updateVariation(uniqueId: string, data: UpdateVariationRequest): Promise<ProductVariation>;
deleteVariation(uniqueId: string): Promise<void>;
listImages(productUniqueId: string): Promise<ProductImage[]>;
addImage(productUniqueId: string, imageUrl: string, isPrimary?: boolean): Promise<ProductImage>;
deleteImage(uniqueId: string): Promise<void>;
getStock(productUniqueId: string, vendorUniqueId?: string): Promise<ProductStock[]>;
updateStock(productUniqueId: string, vendorUniqueId: string, warehouseUniqueId: string, quantity: number): Promise<ProductStock>;
listReviews(productUniqueId: string): Promise<PageResult<ProductReview>>;
addReview(productUniqueId: string, rating: number, title?: string, content?: string): Promise<ProductReview>;

// ProductImagesService — client.products.images
presign(productUniqueId: string): Promise<PresignResponse>;
multipartPresign(productUniqueId: string, filename: string, contentType: string, totalParts: number): Promise<{ uploadId: string; urls: string[] }>;
multipartComplete(productUniqueId: string, uploadId: string, key: string, parts: { etag: string; partNumber: number }[]): Promise<void>;
create(productUniqueId: string, data: CreateProductImageRequest): Promise<ProductImage>;
get(productUniqueId: string, imageUniqueId: string): Promise<ProductImage>;
update(productUniqueId: string, imageUniqueId: string, data: { isPrimary?: boolean }): Promise<ProductImage>;
delete(productUniqueId: string, imageUniqueId: string): Promise<void>;
approve(productUniqueId: string, imageUniqueId: string): Promise<ProductImage>;
publish(productUniqueId: string, imageUniqueId: string): Promise<ProductImage>;

// ProductSuggestionsService — client.products.suggestions
list(productUniqueId: string): Promise<Product[]>;
add(productUniqueId: string, suggestedProductUniqueId: string): Promise<void>;
remove(productUniqueId: string, suggestedProductUniqueId: string): Promise<void>;
getReplacements(productUniqueId: string): Promise<Product[]>;
addReplacements(productUniqueId: string, replacementProductUniqueIds: string[]): Promise<void>;

// AddonsService — client.products.addons
list(productUniqueId: string): Promise<Product[]>;
add(productUniqueId: string, addonProductUniqueId: string): Promise<void>;
remove(productUniqueId: string, addonUniqueId: string): Promise<void>;

// ProductFiltersService — client.products.filters
list(params?: ListProductFiltersParams): Promise<PageResult<ProductFilter>>;
get(uniqueId: string): Promise<ProductFilter>;
create(data: CreateProductFilterRequest): Promise<ProductFilter>;
update(uniqueId: string, data: UpdateProductFilterRequest): Promise<ProductFilter>;
delete(uniqueId: string): Promise<void>;
reorder(filterIds: string[]): Promise<void>;

// ProductVendorsService — client.products.productVendors
listByProduct(productUniqueId: string, params?: ListProductVendorsParams): Promise<PageResult<ProductVendor>>;
getPrimaryVendor(productUniqueId: string): Promise<ProductVendor | null>;
getAvailableVendors(productUniqueId: string, params?: ListProductVendorsParams): Promise<PageResult<ProductVendor>>;
```

### TypeScript Types

```typescript
import type {
  Product,
  ProductVariation,
  ProductImage,
  ProductStock,
  ProductReview,
  CreateProductRequest,
  UpdateProductRequest,
  ListProductsParams,
  CreateVariationRequest,
  UpdateVariationRequest,
  ProductFilter,
  CreateProductFilterRequest,
  UpdateProductFilterRequest,
  ListProductFiltersParams,
  ProductVendor,
  ListProductVendorsParams,
} from '@23blocks/block-products';
```

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: list products with pagination
  const result = await client.products.products.list({ page: 1, perPage: 20 });
}
```
