---
name: products-reviews-api
description: Manage 23blocks product reviews via REST API. Use when creating, updating, deleting, or flagging product reviews, or managing variation-level reviews.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Reviews API

Complete API reference for 23blocks product review management with flagging and variation reviews.

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
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/reviews" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Product Review Endpoints

### GET /products/:unique_id/reviews - List Product Reviews

Lists all reviews for a product.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/reviews" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "review-uuid-123",
      "type": "review",
      "attributes": {
        "unique_id": "review-uuid-123",
        "product_unique_id": "product-uuid-123",
        "user_unique_id": "user-uuid-456",
        "rating": 5,
        "title": "Excellent product!",
        "body": "Really happy with this purchase. Great quality.",
        "status": "published",
        "flagged": false,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "total_count": 24,
    "average_rating": 4.3
  }
}
```

---

### POST /products/:unique_id/reviews - Create Review

Creates a new product review.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/reviews" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "review": {
      "rating": 4,
      "title": "Good product",
      "body": "Works well, minor issues with packaging."
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `rating` | integer | Yes | Rating (1-5) |
| `title` | string | No | Review title |
| `body` | string | Yes | Review body text |

**Response 201:**
```json
{
  "data": {
    "id": "new-review-uuid",
    "type": "review",
    "attributes": {
      "unique_id": "new-review-uuid",
      "product_unique_id": "product-uuid-123",
      "rating": 4,
      "title": "Good product",
      "body": "Works well, minor issues with packaging.",
      "status": "published",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /products/:unique_id/reviews/:review_unique_id - Update Review

Updates an existing review.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/reviews/review-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "review": {
      "rating": 5,
      "body": "Updated review after further use. Excellent!"
    }
  }'
```

**Response 200:** Updated review object

---

### DELETE /products/:unique_id/reviews/:review_unique_id - Delete Review

Deletes a product review.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/products/product-uuid-123/reviews/review-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /products/:unique_id/reviews/:review_unique_id/flag - Flag Review

Flags a review for moderation.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/reviews/review-uuid-123/flag" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "flag": {
      "reason": "inappropriate_content",
      "details": "Contains offensive language"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reason` | string | Yes | Flag reason |
| `details` | string | No | Additional details |

**Response 200:**
```json
{
  "data": {
    "id": "review-uuid-123",
    "type": "review",
    "attributes": {
      "unique_id": "review-uuid-123",
      "flagged": true,
      "flag_reason": "inappropriate_content",
      "status": "flagged"
    }
  }
}
```

---

## Variation Review Endpoints

### GET /products/:unique_id/variations/:variation_unique_id/reviews - List Variation Reviews

Lists all reviews for a product variation.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/variations/variation-uuid-001/reviews" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "vreview-uuid-123",
      "type": "variation_review",
      "attributes": {
        "unique_id": "vreview-uuid-123",
        "variation_unique_id": "variation-uuid-001",
        "user_unique_id": "user-uuid-789",
        "rating": 4,
        "title": "Good variant",
        "body": "The large black version is great.",
        "status": "published",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "total_count": 8,
    "average_rating": 4.1
  }
}
```

---

### POST /products/:unique_id/variations/:variation_unique_id/reviews - Create Variation Review

Creates a review for a product variation.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/variations/variation-uuid-001/reviews" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "review": {
      "rating": 5,
      "title": "Perfect size",
      "body": "The large variant fits perfectly."
    }
  }'
```

**Response 201:** Created variation review object

---

### PUT /products/:unique_id/variations/:variation_unique_id/reviews/:review_unique_id - Update Variation Review

Updates a variation review.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/variations/variation-uuid-001/reviews/vreview-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "review": {
      "rating": 3,
      "body": "Updated after more use."
    }
  }'
```

**Response 200:** Updated variation review object

---

### DELETE /products/:unique_id/variations/:variation_unique_id/reviews/:review_unique_id - Delete Variation Review

Deletes a variation review.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/products/product-uuid-123/variations/variation-uuid-001/reviews/vreview-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Review
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `product_unique_id` | uuid | Product identifier |
| `user_unique_id` | uuid | Reviewer user ID |
| `rating` | integer | Rating (1-5) |
| `title` | string | Review title |
| `body` | string | Review body text |
| `status` | enum | published, flagged, removed |
| `flagged` | boolean | Whether review is flagged |
| `flag_reason` | string | Reason for flagging |
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
    "detail": "Rating must be between 1 and 5."
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
// ProductReviewsService — client.products.reviews
list(productUniqueId: string, page?: number, perPage?: number): Promise<PageResult<ProductReview>>;
create(productUniqueId: string, data: CreateReviewRequest): Promise<ProductReview>;
update(productUniqueId: string, reviewUniqueId: string, data: UpdateReviewRequest): Promise<ProductReview>;
delete(productUniqueId: string, reviewUniqueId: string): Promise<void>;
flag(productUniqueId: string, reviewUniqueId: string): Promise<ProductReview>;
listByUser(userUniqueId: string, page?: number, perPage?: number): Promise<PageResult<ProductReview>>;

// ProductVariationReviewsService — client.products.variationReviews
list(variationUniqueId: string, params?: ListVariationReviewsParams): Promise<PageResult<ProductVariationReview>>;
get(variationUniqueId: string, reviewUniqueId: string): Promise<ProductVariationReview>;
create(variationUniqueId: string, data: CreateVariationReviewRequest): Promise<ProductVariationReview>;
update(variationUniqueId: string, reviewUniqueId: string, data: UpdateVariationReviewRequest): Promise<ProductVariationReview>;
delete(variationUniqueId: string, reviewUniqueId: string): Promise<void>;
markHelpful(variationUniqueId: string, reviewUniqueId: string): Promise<ProductVariationReview>;
markNotHelpful(variationUniqueId: string, reviewUniqueId: string): Promise<ProductVariationReview>;
flag(variationUniqueId: string, reviewUniqueId: string): Promise<ProductVariationReview>;
listByUser(userUniqueId: string, params?: ListVariationReviewsParams): Promise<PageResult<ProductVariationReview>>;
getAverageRating(variationUniqueId: string): Promise<{ averageRating: number; totalReviews: number }>;
```

### TypeScript Types

```typescript
import type {
  ProductReview,
  ProductVariationReview,
  CreateVariationReviewRequest,
  UpdateVariationReviewRequest,
  ListVariationReviewsParams,
} from '@23blocks/block-products';
```

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: list reviews for a product
  const result = await client.products.reviews.list('product-uuid-123');
}
```
