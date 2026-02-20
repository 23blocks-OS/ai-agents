---
name: products-categories-api
description: Manage 23blocks product categories via REST API. Use when creating, updating, deleting, or recovering product categories, including cascade deletion and category images.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Categories API

Complete API reference for 23blocks product category management with images and hierarchy.

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
curl -X GET "$BLOCKS_API_URL/categories/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /categories/ - List Categories

Lists all product categories.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/categories/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "category-uuid-123",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-123",
        "name": "Electronics",
        "description": "Electronic devices and accessories",
        "parent_id": null,
        "products_count": 156,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /categories/:unique_id/ - Get Category

Retrieves a single category by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/categories/category-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "category-uuid-123",
    "type": "category",
    "attributes": {
      "unique_id": "category-uuid-123",
      "name": "Electronics",
      "description": "Electronic devices and accessories",
      "parent_id": null,
      "products_count": 156,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "children": {
        "data": [
          { "id": "category-uuid-456", "type": "category" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Category not found

---

### POST /categories/ - Create Category

Creates a new product category.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/categories/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Keyboards",
      "description": "Computer keyboards",
      "parent_id": "electronics-uuid"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Category name |
| `description` | string | No | Category description |
| `parent_id` | uuid | No | Parent category ID (null for root) |

**Response 201:**
```json
{
  "data": {
    "id": "new-category-uuid",
    "type": "category",
    "attributes": {
      "unique_id": "new-category-uuid",
      "name": "Keyboards",
      "description": "Computer keyboards",
      "parent_id": "electronics-uuid",
      "products_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /categories/:unique_id - Update Category

Updates an existing category.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/categories/category-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Updated Category Name",
      "description": "Updated description"
    }
  }'
```

**Response 200:** Updated category object

---

### DELETE /categories/:unique_id - Delete Category

Soft-deletes a category.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/categories/category-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### DELETE /categories/:unique_id/cascade - Cascade Delete

Deletes a category and all its child categories.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/categories/category-uuid-123/cascade" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /categories/:unique_id/recover - Recover Category

Recovers a soft-deleted category.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/categories/category-uuid-123/recover" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "category-uuid-123",
    "type": "category",
    "attributes": {
      "unique_id": "category-uuid-123",
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Category Images

### PUT /categories/:unique_id/presign - Get Presigned URL

Gets a presigned URL for category image upload.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/categories/category-uuid-123/presign" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file_name": "category-image.jpg",
    "content_type": "image/jpeg"
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "presigned_url",
    "attributes": {
      "upload_url": "https://s3.amazonaws.com/bucket/presigned-upload-url",
      "file_url": "https://cdn.example.com/categories/category-image.jpg",
      "expires_in": 3600
    }
  }
}
```

---

### POST /categories/:unique_id/images - Upload Image

Registers an uploaded image to a category.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/categories/category-uuid-123/images" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image": {
      "file_url": "https://cdn.example.com/categories/category-image.jpg",
      "alt_text": "Electronics category"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "image-uuid-123",
    "type": "category_image",
    "attributes": {
      "unique_id": "image-uuid-123",
      "file_url": "https://cdn.example.com/categories/category-image.jpg",
      "status": "pending",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /categories/:unique_id/images/:unique_file_id - Delete Image

Deletes a category image.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/categories/category-uuid-123/images/image-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /categories/:unique_id/images/:unique_file_id/approve - Approve Image

Approves a category image.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/categories/category-uuid-123/images/image-uuid-123/approve" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "image-uuid-123",
    "type": "category_image",
    "attributes": {
      "status": "approved"
    }
  }
}
```

---

### PUT /categories/:unique_id/images/:unique_file_id/publish - Publish Image

Publishes an approved category image.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/categories/category-uuid-123/images/image-uuid-123/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "image-uuid-123",
    "type": "category_image",
    "attributes": {
      "status": "published"
    }
  }
}
```

---

## Data Models

### Category
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Category name |
| `description` | string | Category description |
| `parent_id` | uuid | Parent category ID (null for root) |
| `products_count` | integer | Number of products |
| `status` | enum | active, inactive, trash |
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
// CategoriesService — client.products.categories
list(params?: ListCategoriesParams): Promise<PageResult<Category>>;
get(uniqueId: string): Promise<Category>;
create(data: CreateCategoryRequest): Promise<Category>;
update(uniqueId: string, data: UpdateCategoryRequest): Promise<Category>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<Category>;
getChildren(uniqueId: string): Promise<Category[]>;
```

### TypeScript Types

```typescript
import type {
  Category,
  CreateCategoryRequest,
  UpdateCategoryRequest,
  ListCategoriesParams,
} from '@23blocks/block-products';
```

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: list categories with children
  const result = await client.products.categories.list({ withChildren: true });
}
```
