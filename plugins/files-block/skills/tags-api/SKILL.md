---
name: files-tags-api
description: Create and manage file tags via REST API. Use tags to add flexible metadata and enable powerful file search and filtering.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Tags API

Complete API reference for 23blocks file tag management.

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
curl -X GET "$BLOCKS_API_URL/tags" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Tag Management Endpoints

### GET /tags - List Tags

Lists all tags for the tenant.

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
      "id": "tag-123",
      "type": "tag",
      "attributes": {
        "unique_id": "tag-123",
        "tag": "legal",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "tag-456",
      "type": "tag",
      "attributes": {
        "unique_id": "tag-456",
        "tag": "2025",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /tags/:unique_id - Get Tag

Retrieves a single tag by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tags/$TAG_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "tag-123",
    "type": "tag",
    "attributes": {
      "unique_id": "tag-123",
      "tag": "legal",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### POST /tags - Create Tag

Creates a new tag.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/tags" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag": {
      "tag": "confidential"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tag` | string | Yes | Tag name |

**Response 201:**
```json
{
  "data": {
    "id": "new-tag-id",
    "type": "tag",
    "attributes": {
      "unique_id": "new-tag-id",
      "tag": "confidential",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### PUT /tags/:unique_id - Update Tag

Updates an existing tag.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/tags/$TAG_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag": {
      "tag": "highly-confidential"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "tag-id",
    "type": "tag",
    "attributes": {
      "tag": "highly-confidential",
      "updated_at": "2025-01-10T14:00:00Z"
    }
  }
}
```

---

## File Tag Endpoints

### POST /users/:unique_id/files/:unique_file_id/tags - Add Tags to File

Adds tags to a specific file.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/tags" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tags": ["legal", "2025", "contract"]
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tags` | array | Yes | Array of tag names |

**Response 200:**
```json
{
  "data": {
    "id": "file-id",
    "type": "user_file",
    "attributes": {
      "tags": ["legal", "2025", "contract"]
    }
  }
}
```

**Note:** Tags are automatically created if they don't exist.

---

### DELETE /users/:unique_id/files/:unique_file_id/tags - Remove Tags from File

Removes specific tags from a file.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/tags" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tags": ["2025"]
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "file-id",
    "type": "user_file",
    "attributes": {
      "tags": ["legal", "contract"]
    }
  }
}
```

---

### POST /users/:unique_id/tags - Bulk Update Tags

Updates tags for multiple files at once.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/tags" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file_unique_ids": ["file-1", "file-2", "file-3"],
    "tags_to_add": ["project-x", "2025"],
    "tags_to_remove": ["draft"]
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file_unique_ids` | array | Yes | Files to update |
| `tags_to_add` | array | No | Tags to add |
| `tags_to_remove` | array | No | Tags to remove |

**Response 200:**
```json
{
  "data": {
    "files_updated": 3,
    "tags_added": ["project-x", "2025"],
    "tags_removed": ["draft"]
  }
}
```

---

## Data Models

### Tag
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `tag` | string | Tag name |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### FileTag (Junction Table)
| Field | Type | Description |
|-------|------|-------------|
| `tag_id` | integer | Tag FK |
| `user_file_id` | integer | File FK |
| `tag_unique_id` | uuid | Tag UUID |
| `user_file_unique_id` | uuid | File UUID |

---

## Filtering Files by Tags

Use the `tags` query parameter when listing files:

```bash
# Filter by single tag
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/files?tags=legal" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"

# Filter by multiple tags (comma-separated)
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/files?tags=legal,2025" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```
