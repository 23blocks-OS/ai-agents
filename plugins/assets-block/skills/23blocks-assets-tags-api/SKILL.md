---
name: 23blocks-assets-tags-api
description: "Assets Block tags: create, list, update. Use when labeling assets."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Tags API

Complete API reference for 23blocks Assets Block tag management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://assets.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /tags/ - List Tags

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
        "tag": "high-value",
        "thumbnail_url": "https://example.com/high-value.png",
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
        "tag": "warranty-active",
        "thumbnail_url": null,
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
      "tag": "high-value",
      "thumbnail_url": "https://example.com/high-value.png",
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag": {
      "tag": "fragile",
      "thumbnail_url": "https://example.com/fragile-icon.png"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tag` | string | Yes | Tag value (unique) |
| `thumbnail_url` | string | No | Thumbnail image URL for the tag |

**Response 201:**
```json
{
  "data": {
    "id": "new-tag-uuid",
    "type": "tag",
    "attributes": {
      "unique_id": "new-tag-uuid",
      "tag": "fragile",
      "thumbnail_url": "https://example.com/fragile-icon.png",
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag": {
      "tag": "premium",
      "thumbnail_url": "https://example.com/premium-icon.png"
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
      "tag": "premium",
      "thumbnail_url": "https://example.com/premium-icon.png",
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
| `tag` | string | Tag value (unique) |
| `thumbnail_url` | string | Thumbnail image URL |
| `assets_count` | integer | Number of assets with this tag |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Error","detail":"Name has already been taken."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-assets`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useAssetsBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// TagsService — client.assets.tags
client.assets.tags.list(params?: ListTagsParams): Promise<PageResult<Tag>>;
client.assets.tags.get(uniqueId: string): Promise<Tag>;
client.assets.tags.create(data: CreateTagRequest): Promise<Tag>;
client.assets.tags.update(uniqueId: string, data: UpdateTagRequest): Promise<Tag>;
```

### TypeScript Types

```typescript
import type {
  Tag,
  CreateTagRequest,
  UpdateTagRequest,
  ListTagsParams,
} from '@23blocks/block-assets';
```
