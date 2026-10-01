---
name: 23blocks-files-tags-api
description: "Files Block tags on files. Use when tagging files for search and filters."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Tags API

Complete API reference for 23blocks file tag management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://files.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Tag Management Endpoints

### GET /tags - List Tags

Lists all tags for the tenant.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tags" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag": {
      "tag": "confidential",
      "description": "Files marked as confidential"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tag` | string | Yes | Tag name |
| `description` | string | No | Tag description |

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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
| `description` | string | Tag description |
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
  -H "X-API-KEY: $BLOCKS_API_KEY"

# Filter by multiple tags (comma-separated)
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/files?tags=legal,2025" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Failed","detail":"Tag can't be blank."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-files`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useFilesBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// FileTagsService — client.files.fileTags
list(params?: ListFileTagsParams): Promise<PageResult<FileTag>>;
get(uniqueId: string): Promise<FileTag>;
create(data: CreateFileTagRequest): Promise<FileTag>;
update(uniqueId: string, data: UpdateFileTagRequest): Promise<FileTag>;
delete(uniqueId: string): Promise<void>;
addToFile(userUniqueId: string, fileUniqueId: string, tagUniqueId: string): Promise<void>;
removeFromFile(userUniqueId: string, fileUniqueId: string, tagUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  FileTag,
  CreateFileTagRequest,
  UpdateFileTagRequest,
  ListFileTagsParams,
  FileTagAssignment,
} from '@23blocks/block-files';
```
