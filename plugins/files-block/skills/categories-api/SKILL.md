---
name: files-categories-api
description: Create and manage file categories via REST API. Use categories to organize files into logical groups for better file management.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Categories API

Complete API reference for 23blocks file category management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://files.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Files API base URL | `https://files.api.us.23blocks.com` |
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

### GET /categories - List Categories

Lists all categories for the tenant.

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
      "id": "cat-123",
      "type": "category",
      "attributes": {
        "unique_id": "cat-123",
        "name": "Contracts",
        "code": "CONTRACTS",
        "description": "Legal contracts and agreements",
        "status": "active",
        "display_order": 1,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "cat-456",
      "type": "category",
      "attributes": {
        "unique_id": "cat-456",
        "name": "Reports",
        "code": "REPORTS",
        "description": "Monthly and annual reports",
        "status": "active",
        "display_order": 2,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /categories/:unique_id - Get Category

Retrieves a single category by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/categories/$CATEGORY_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "cat-123",
    "type": "category",
    "attributes": {
      "unique_id": "cat-123",
      "name": "Contracts",
      "code": "CONTRACTS",
      "description": "Legal contracts and agreements",
      "status": "active",
      "display_order": 1,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Category not found

---

### POST /categories - Create Category

Creates a new category.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Invoices",
      "code": "INVOICES",
      "description": "Customer and vendor invoices",
      "display_order": 3
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Category display name |
| `code` | string | Yes | Unique category code (uppercase) |
| `description` | string | No | Category description |
| `display_order` | integer | No | Sort order for display |

**Response 201:**
```json
{
  "data": {
    "id": "new-cat-id",
    "type": "category",
    "attributes": {
      "unique_id": "new-cat-id",
      "name": "Invoices",
      "code": "INVOICES",
      "description": "Customer and vendor invoices",
      "status": "active",
      "display_order": 3,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### PUT /categories/:unique_id - Update Category

Updates an existing category.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/categories/$CATEGORY_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Customer Invoices",
      "description": "Updated description",
      "display_order": 5
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "cat-id",
    "type": "category",
    "attributes": {
      "name": "Customer Invoices",
      "description": "Updated description",
      "display_order": 5,
      "updated_at": "2025-01-10T14:00:00Z"
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
| `name` | string | Display name |
| `code` | string | Unique code (uppercase) |
| `description` | string | Description |
| `status` | enum | active, inactive |
| `display_order` | integer | Sort order |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Default Category

If no category is specified when creating a file, the system uses the default "Uncategorized" category:

```json
{
  "name": "Uncategorized",
  "code": "UNCATEGORIZED",
  "description": "Default category for files without a specific category",
  "display_order": 0
}
```

This category is auto-created if it doesn't exist (self-healing for legacy tenants).
