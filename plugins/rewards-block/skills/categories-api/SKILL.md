---
name: rewards-categories-api
description: Manage 23blocks badge categories via REST API. Use when listing, retrieving, or creating categories to organize badges into logical groups.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Categories API

Complete API reference for 23blocks badge category management to organize badges into logical groups.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://rewards.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Rewards API base URL | `https://rewards.api.us.23blocks.com` |
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

Lists all badge categories.

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
      "id": "category-uuid-1",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-1",
        "name": "Shopping",
        "description": "Badges earned through purchases",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "category-uuid-2",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-2",
        "name": "Engagement",
        "description": "Badges earned through community participation",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "category-uuid-3",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-3",
        "name": "Milestones",
        "description": "Badges for reaching account milestones",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
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
curl -X GET "$BLOCKS_API_URL/categories/category-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "category-uuid-1",
    "type": "category",
    "attributes": {
      "unique_id": "category-uuid-1",
      "name": "Shopping",
      "description": "Badges earned through purchases",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "badges": {
        "data": [
          { "id": "badge-uuid-1", "type": "badge" },
          { "id": "badge-uuid-2", "type": "badge" }
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

Creates a new badge category.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Shopping",
      "description": "Badges earned through purchases"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Category name |
| `description` | string | No | Category description |

**Response 201:**
```json
{
  "data": {
    "id": "new-category-uuid",
    "type": "category",
    "attributes": {
      "unique_id": "new-category-uuid",
      "name": "Shopping",
      "description": "Badges earned through purchases",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Category name already exists
- `422 Unprocessable Entity` - Validation errors

---

## Data Models

### Category
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Category name |
| `description` | string | Category description |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "409",
    "code": "duplicate_name",
    "title": "Category Already Exists",
    "detail": "A category with this name already exists."
  }]
}
```
