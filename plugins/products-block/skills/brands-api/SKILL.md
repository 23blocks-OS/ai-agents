---
name: products-brands-api
description: Manage 23blocks product brands via REST API. Use when creating, updating, deleting brands, or viewing trashed brands.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Brands API

Complete API reference for 23blocks product brand management.

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
curl -X GET "$BLOCKS_API_URL/brands/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /brands/ - List Brands

Lists all brands.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/brands/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "brand-uuid-123",
      "type": "brand",
      "attributes": {
        "unique_id": "brand-uuid-123",
        "name": "TechCo",
        "description": "Premium technology products",
        "logo_url": "https://example.com/logo.png",
        "website": "https://techco.com",
        "products_count": 45,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /brands/:unique_id/ - Get Brand

Retrieves a single brand by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/brands/brand-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "brand-uuid-123",
    "type": "brand",
    "attributes": {
      "unique_id": "brand-uuid-123",
      "name": "TechCo",
      "description": "Premium technology products",
      "logo_url": "https://example.com/logo.png",
      "website": "https://techco.com",
      "products_count": 45,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Brand not found

---

### POST /brands/ - Create Brand

Creates a new brand.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/brands/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "brand": {
      "name": "NewBrand",
      "description": "New brand description",
      "logo_url": "https://example.com/newbrand-logo.png",
      "website": "https://newbrand.com"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Brand name |
| `description` | string | No | Brand description |
| `logo_url` | string | No | Logo image URL |
| `website` | string | No | Brand website |

**Response 201:**
```json
{
  "data": {
    "id": "new-brand-uuid",
    "type": "brand",
    "attributes": {
      "unique_id": "new-brand-uuid",
      "name": "NewBrand",
      "description": "New brand description",
      "products_count": 0,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /brands/:unique_id - Update Brand

Updates an existing brand.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/brands/brand-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "brand": {
      "name": "TechCo Premium",
      "description": "Updated brand description"
    }
  }'
```

**Response 200:** Updated brand object

---

### DELETE /brands/:unique_id - Delete Brand

Soft-deletes a brand.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/brands/brand-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### GET /brands/trash/show - View Trash

Lists all soft-deleted brands.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/brands/trash/show" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "brand-uuid-456",
      "type": "brand",
      "attributes": {
        "unique_id": "brand-uuid-456",
        "name": "Old Brand",
        "status": "trash",
        "deleted_at": "2025-01-11T08:00:00Z"
      }
    }
  ]
}
```

---

## Data Models

### Brand
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Brand name |
| `description` | string | Brand description |
| `logo_url` | string | Logo image URL |
| `website` | string | Brand website |
| `products_count` | integer | Number of products |
| `status` | enum | active, inactive, trash |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Brand Not Found",
    "detail": "The requested brand could not be found."
  }]
}
```
