---
name: 23blocks-content-tags-api
description: "Content Block tags: create, list, update. Use when tagging posts."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Tags API

Complete API reference for 23blocks tag management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://content.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /tags - List Tags

Lists all tags.

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
      "id": "tag-uuid-123",
      "type": "tag",
      "attributes": {
        "unique_id": "tag-uuid-123",
        "name": "technology",
        "description": "Technology related content",
        "posts_count": 42,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "tag-uuid-456",
      "type": "tag",
      "attributes": {
        "unique_id": "tag-uuid-456",
        "name": "tutorial",
        "description": "How-to guides and tutorials",
        "posts_count": 28,
        "created_at": "2025-01-08T15:00:00Z",
        "updated_at": "2025-01-08T15:00:00Z"
      }
    }
  ]
}
```

---

### GET /tags/:unique_id - Get Tag

Retrieves a single tag by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tags/tag-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "tag-uuid-123",
    "type": "tag",
    "attributes": {
      "unique_id": "tag-uuid-123",
      "name": "technology",
      "description": "Technology related content",
      "posts_count": 42,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Tag not found

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
      "name": "programming",
      "description": "Programming and coding content"
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
      "name": "programming",
      "description": "Programming and coding content",
      "posts_count": 0,
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag": {
      "name": "tech",
      "description": "Updated technology tag description"
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
      "name": "tech",
      "description": "Updated technology tag description",
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
| `posts_count` | integer | Number of posts with this tag |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Error","detail":"Name has already been taken."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-content`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useContentBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// TagsService — client.content.tags
list(params?: ListTagsParams): Promise<PageResult<Tag>>;
get(uniqueId: string): Promise<Tag>;
create(data: CreateTagRequest): Promise<Tag>;
update(uniqueId: string, data: UpdateTagRequest): Promise<Tag>;
delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Tag,
  CreateTagRequest,
  UpdateTagRequest,
  ListTagsParams,
} from '@23blocks/block-content';
```
