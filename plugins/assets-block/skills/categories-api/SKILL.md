---
name: assets-categories-api
description: Create and manage 23blocks asset categories via REST API. Use when creating, listing, updating, or deleting categories for asset organization, including image management and cascade deletion.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Categories API

Complete API reference for 23blocks Assets Block category management with image support and cascade deletion.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://assets.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Assets API base URL | `https://assets.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /categories/ - List Categories

Lists all categories.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/categories" \
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
        "description": "Electronic devices and equipment",
        "parent_id": null,
        "assets_count": 45,
        "image_url": "https://example.com/electronics.jpg",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "category-uuid-456",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-456",
        "name": "Laptops",
        "description": "Laptop computers",
        "parent_id": "category-uuid-123",
        "assets_count": 20,
        "image_url": null,
        "created_at": "2025-01-08T15:00:00Z",
        "updated_at": "2025-01-08T15:00:00Z"
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
curl -X GET "$BLOCKS_API_URL/categories/category-uuid-123" \
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
      "description": "Electronic devices and equipment",
      "parent_id": null,
      "assets_count": 45,
      "image_url": "https://example.com/electronics.jpg",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
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

Creates a new category.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Furniture",
      "description": "Office furniture and fixtures",
      "parent_id": null
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
      "name": "Furniture",
      "description": "Office furniture and fixtures",
      "parent_id": null,
      "assets_count": 0,
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
      "name": "Updated Electronics",
      "description": "Updated description"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "category-uuid-123",
    "type": "category",
    "attributes": {
      "unique_id": "category-uuid-123",
      "name": "Updated Electronics",
      "description": "Updated description",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Category not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /categories/:unique_id - Delete Category

Deletes a category. Fails if the category has children.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/categories/category-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Category not found
- `422 Unprocessable Entity` - Category has children (use cascade delete)

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

**Errors:**
- `404 Not Found` - Category not found

---

## Category Images

### PUT /categories/:unique_id/presign - Presign Image Upload

Generates a presigned URL for uploading a category image.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/categories/category-uuid-123/presign" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "filename": "category-image.jpg",
      "content_type": "image/jpeg"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filename` | string | Yes | Name of the file |
| `content_type` | string | Yes | MIME type (e.g., image/jpeg, image/png) |

**Response 200:**
```json
{
  "data": {
    "presigned_url": "https://storage.example.com/upload?signature=abc123",
    "file_unique_id": "file-uuid-789",
    "expires_at": "2025-01-12T11:30:00Z"
  }
}
```

---

### POST /categories/:unique_id/images - Upload Image

Confirms an image upload after using the presigned URL.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/categories/category-uuid-123/images" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "file_unique_id": "file-uuid-789"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "file-uuid-789",
    "type": "image",
    "attributes": {
      "unique_id": "file-uuid-789",
      "url": "https://storage.example.com/categories/category-image.jpg",
      "filename": "category-image.jpg",
      "content_type": "image/jpeg",
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
curl -X DELETE "$BLOCKS_API_URL/categories/category-uuid-123/images/file-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Category
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Category name |
| `description` | string | Category description |
| `parent_id` | uuid | Parent category ID (null for root) |
| `assets_count` | integer | Number of assets in category |
| `image_url` | string | Category image URL |
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
