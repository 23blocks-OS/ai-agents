---
name: assets-tags-api
description: Create and manage 23blocks asset tags via REST API. Use when creating, listing, or updating tags for asset labeling and organization.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Tags API

Complete API reference for 23blocks Assets Block tag management.

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
curl -X GET "$BLOCKS_API_URL/tags" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /tags/ - List Tags

Lists all tags.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tags" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "tag-uuid-123",
      "type": "tag",
      "attributes": {
        "unique_id": "tag-uuid-123",
        "name": "high-value",
        "description": "High-value assets",
        "assets_count": 18,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "tag-uuid-456",
      "type": "tag",
      "attributes": {
        "unique_id": "tag-uuid-456",
        "name": "warranty-active",
        "description": "Assets with active warranty",
        "assets_count": 32,
        "created_at": "2025-01-08T15:00:00Z",
        "updated_at": "2025-01-08T15:00:00Z"
      }
    }
  ]
}
```

---

### GET /tags/:unique_id/ - Get Tag

Retrieves a single tag by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tags/tag-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "tag-uuid-123",
    "type": "tag",
    "attributes": {
      "unique_id": "tag-uuid-123",
      "name": "high-value",
      "description": "High-value assets",
      "assets_count": 18,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Tag not found

---

### POST /tags/ - Create Tag

Creates a new tag.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/tags" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag": {
      "name": "fragile",
      "description": "Fragile assets requiring careful handling"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Tag name (unique) |
| `description` | string | No | Tag description |

**Response 201:**
```json
{
  "data": {
    "id": "new-tag-uuid",
    "type": "tag",
    "attributes": {
      "unique_id": "new-tag-uuid",
      "name": "fragile",
      "description": "Fragile assets requiring careful handling",
      "assets_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (e.g., name already exists)

---

### PUT /tags/:unique_id - Update Tag

Updates an existing tag.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/tags/tag-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag": {
      "name": "premium",
      "description": "Updated premium asset tag"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "tag-uuid-123",
    "type": "tag",
    "attributes": {
      "unique_id": "tag-uuid-123",
      "name": "premium",
      "description": "Updated premium asset tag",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Tag not found
- `422 Unprocessable Entity` - Validation errors

---

## Data Models

### Tag
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Tag name (unique) |
| `description` | string | Tag description |
| `assets_count` | integer | Number of assets with this tag |
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
    "detail": "Name has already been taken."
  }]
}
```
