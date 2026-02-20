---
name: products-products-api
description: Manage 23blocks products via REST API. Use when creating, updating, searching products, managing product images, assigning categories and catalogs, handling suggestions, addons, replacements, and product filters.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Products API

Complete API reference for 23blocks product management with images, associations, and search.

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
curl -X GET "$BLOCKS_API_URL/products/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /products/ - List Products

Lists all products with pagination and search.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `search` | string | No | Search term |

**Response 200:**
```json
{
  "data": [
    {
      "id": "product-uuid-123",
      "type": "product",
      "attributes": {
        "unique_id": "product-uuid-123",
        "name": "Wireless Keyboard",
        "description": "Bluetooth wireless keyboard with backlight",
        "sku": "KB-001",
        "barcode": "1234567890123",
        "brand_id": "brand-uuid",
        "category_id": "category-uuid",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### GET /products/:unique_id/ - Get Product

Retrieves a single product by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "product-uuid-123",
    "type": "product",
    "attributes": {
      "unique_id": "product-uuid-123",
      "name": "Wireless Keyboard",
      "description": "Bluetooth wireless keyboard with backlight",
      "sku": "KB-001",
      "barcode": "1234567890123",
      "brand_id": "brand-uuid",
      "category_id": "category-uuid",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "brand": {
        "data": { "id": "brand-uuid", "type": "brand" }
      },
      "category": {
        "data": { "id": "category-uuid", "type": "category" }
      },
      "images": {
        "data": []
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Product not found

---

### POST /products/ - Create Product

Creates a new product.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "product": {
      "name": "Wireless Mouse",
      "description": "Ergonomic wireless mouse",
      "sku": "MS-001",
      "barcode": "9876543210123",
      "brand_id": "brand-uuid",
      "category_id": "category-uuid",
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Product name |
| `description` | string | No | Product description |
| `sku` | string | No | Stock keeping unit |
| `barcode` | string | No | Product barcode |
| `brand_id` | uuid | No | Brand identifier |
| `category_id` | uuid | No | Category identifier |
| `status` | enum | No | active, inactive (default: active) |

**Response 201:**
```json
{
  "data": {
    "id": "new-product-uuid",
    "type": "product",
    "attributes": {
      "unique_id": "new-product-uuid",
      "name": "Wireless Mouse",
      "description": "Ergonomic wireless mouse",
      "sku": "MS-001",
      "barcode": "9876543210123",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /products/:unique_id - Update Product

Updates an existing product.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "product": {
      "name": "Premium Wireless Keyboard",
      "description": "Updated description"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "product-uuid-123",
    "type": "product",
    "attributes": {
      "unique_id": "product-uuid-123",
      "name": "Premium Wireless Keyboard",
      "description": "Updated description",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /products/:unique_id - Delete Product

Soft-deletes a product (moves to trash).

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/products/product-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /products/:unique_id/recover - Recover Product

Recovers a soft-deleted product from trash.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/recover" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "product-uuid-123",
    "type": "product",
    "attributes": {
      "unique_id": "product-uuid-123",
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### GET /products/trash/show - View Trash

Lists all soft-deleted products.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/trash/show" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "product-uuid-456",
      "type": "product",
      "attributes": {
        "unique_id": "product-uuid-456",
        "name": "Deleted Product",
        "status": "trash",
        "deleted_at": "2025-01-11T08:00:00Z"
      }
    }
  ]
}
```

---

### GET /catalog/ - Catalog View

Returns a catalog-optimized view of products.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/catalog/?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "product-uuid-123",
      "type": "product",
      "attributes": {
        "unique_id": "product-uuid-123",
        "name": "Wireless Keyboard",
        "sku": "KB-001",
        "status": "active",
        "primary_image_url": "https://example.com/image.jpg",
        "price": 49.99,
        "currency": "USD"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### POST /products/search - Search Products

Full-text search across products.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "search": {
      "query": "wireless keyboard",
      "category_id": "category-uuid",
      "brand_id": "brand-uuid",
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Search query |
| `category_id` | uuid | No | Filter by category |
| `brand_id` | uuid | No | Filter by brand |
| `status` | enum | No | Filter by status |

**Response 200:** Same format as GET /products/

---

### GET /products/:unique_id/replacements - List Replacements

Lists replacement products for a given product.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/replacements" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "product-uuid-789",
      "type": "product",
      "attributes": {
        "unique_id": "product-uuid-789",
        "name": "Replacement Keyboard V2",
        "sku": "KB-002",
        "status": "active"
      }
    }
  ]
}
```

---

## Product Tools

### GET /tools/products/payload/ - Payload Tool

Returns the payload structure for product operations.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tools/products/payload/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "payload",
    "attributes": {
      "fields": ["name", "description", "sku", "barcode", "brand_id", "category_id", "status"]
    }
  }
}
```

---

### GET /tools/products/payload/filters - Payload Filters

Returns available filter fields for product payloads.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tools/products/payload/filters" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "payload_filters",
    "attributes": {
      "filters": ["category_id", "brand_id", "status", "sku", "barcode"]
    }
  }
}
```

---

### GET /tools/products/filters - List Filters

Lists saved product filters.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tools/products/filters" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "filter-uuid-123",
      "type": "product_filter",
      "attributes": {
        "unique_id": "filter-uuid-123",
        "name": "Active Electronics",
        "criteria": {"category_id": "electronics-uuid", "status": "active"},
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /tools/products/filters - Create Filter

Creates a new product filter.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/tools/products/filters" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "name": "Active Electronics",
      "criteria": {"category_id": "electronics-uuid", "status": "active"}
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "new-filter-uuid",
    "type": "product_filter",
    "attributes": {
      "unique_id": "new-filter-uuid",
      "name": "Active Electronics",
      "criteria": {"category_id": "electronics-uuid", "status": "active"},
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /tools/products/filters/:unique_id - Update Filter

Updates an existing product filter.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/tools/products/filters/filter-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "name": "Updated Filter Name"
    }
  }'
```

**Response 200:** Updated filter object

---

### DELETE /tools/products/filters/:unique_id - Delete Filter

Deletes a product filter.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/tools/products/filters/filter-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Product Images

### PUT /products/:unique_id/presign - Get Presigned URL

Gets a presigned URL for image upload.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/presign" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file_name": "product-image.jpg",
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
      "file_url": "https://cdn.example.com/products/product-image.jpg",
      "expires_in": 3600
    }
  }
}
```

---

### POST /products/:unique_id/images - Upload Image

Registers an uploaded image to a product.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/images" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image": {
      "file_url": "https://cdn.example.com/products/product-image.jpg",
      "alt_text": "Product front view",
      "position": 1
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "image-uuid-123",
    "type": "product_image",
    "attributes": {
      "unique_id": "image-uuid-123",
      "file_url": "https://cdn.example.com/products/product-image.jpg",
      "alt_text": "Product front view",
      "position": 1,
      "status": "pending",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### GET /products/:unique_id/images - List Images

Lists all images for a product.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/images" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "image-uuid-123",
      "type": "product_image",
      "attributes": {
        "unique_id": "image-uuid-123",
        "file_url": "https://cdn.example.com/products/product-image.jpg",
        "alt_text": "Product front view",
        "position": 1,
        "status": "published"
      }
    }
  ]
}
```

---

### PUT /products/:unique_id/images/:unique_file_id - Update Image

Updates image metadata.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/images/image-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image": {
      "alt_text": "Updated alt text",
      "position": 2
    }
  }'
```

**Response 200:** Updated image object

---

### DELETE /products/:unique_id/images/:unique_file_id - Delete Image

Deletes a product image.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/products/product-uuid-123/images/image-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /products/:unique_id/images/:unique_file_id/approve - Approve Image

Approves a product image for use.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/images/image-uuid-123/approve" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "image-uuid-123",
    "type": "product_image",
    "attributes": {
      "status": "approved"
    }
  }
}
```

---

### PUT /products/:unique_id/images/:unique_file_id/publish - Publish Image

Publishes an approved product image.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/images/image-uuid-123/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "image-uuid-123",
    "type": "product_image",
    "attributes": {
      "status": "published"
    }
  }
}
```

---

## Product Categories & Catalogs

### POST /products/:unique_id/categories - Assign Category

Assigns a category to a product.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category_unique_id": "category-uuid-456"
  }'
```

**Response 200:**
```json
{
  "message": "Category assigned successfully"
}
```

---

### DELETE /products/:unique_id/categories/:category_unique_id - Remove Category

Removes a category from a product.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/products/product-uuid-123/categories/category-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Category removed successfully"
}
```

---

### POST /products/:unique_id/catalogs - Assign to Catalog

Assigns a product to a catalog.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/catalogs" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "catalog_unique_id": "catalog-uuid-789"
  }'
```

**Response 200:**
```json
{
  "message": "Product assigned to catalog successfully"
}
```

---

## Suggestions, Addons & Replacements

### GET /products/:unique_id/suggestions - List Suggestions

Lists suggested products.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/suggestions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "product-uuid-789",
      "type": "product",
      "attributes": {
        "unique_id": "product-uuid-789",
        "name": "Suggested Product",
        "sku": "SP-001"
      }
    }
  ]
}
```

---

### POST /products/:unique_id/suggestions - Add Suggestion

Adds a product as a suggestion.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/suggestions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "product_unique_id": "product-uuid-789"
  }'
```

**Response 200:**
```json
{
  "message": "Suggestion added successfully"
}
```

---

### DELETE /products/:unique_id/suggestions/:product_unique_id - Remove Suggestion

Removes a product suggestion.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/products/product-uuid-123/suggestions/product-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Suggestion removed successfully"
}
```

---

### GET /products/:unique_id/addons - List Addons

Lists product addons.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/addons" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "addon-uuid-456",
      "type": "addon",
      "attributes": {
        "unique_id": "addon-uuid-456",
        "name": "Extended Warranty",
        "price": 19.99
      }
    }
  ]
}
```

---

### POST /products/:unique_id/addons - Add Addon

Adds an addon to a product.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/addons" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "addon_unique_id": "addon-uuid-456"
  }'
```

**Response 200:**
```json
{
  "message": "Addon added successfully"
}
```

---

### DELETE /products/:unique_id/addons/:addon_unique_id - Remove Addon

Removes an addon from a product.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/products/product-uuid-123/addons/addon-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Addon removed successfully"
}
```

---

### POST /products/:unique_id/replacements - Add Replacement

Adds a replacement product.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/replacements" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "product_unique_id": "product-uuid-789"
  }'
```

**Response 200:**
```json
{
  "message": "Replacement added successfully"
}
```

---

## Product Vendors

### GET /products/:unique_id/vendors - List Product Vendors

Lists vendors associated with a product.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/vendors" \
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
        "status": "active"
      }
    }
  ]
}
```

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
