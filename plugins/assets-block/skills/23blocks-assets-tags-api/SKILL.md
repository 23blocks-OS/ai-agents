---
name: 23blocks-assets-tags-api
description: Create and manage 23blocks asset tags via REST API. Use when creating, listing, or updating tags for asset labeling and organization.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Tags API

Complete API reference for 23blocks Assets Block tag management.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Assets API base URL | `https://assets.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://assets.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://assets.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

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

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-assets
```

### Setup

```typescript
import { create23BlocksClient } from '@23blocks/sdk';

const client = create23BlocksClient({
  authToken: process.env.BLOCKS_AUTH_TOKEN!,
  apiKey: process.env.BLOCKS_API_KEY!,
  apiUrl: process.env.BLOCKS_API_URL!,
});
```

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

### React Hook

```typescript
import { useAssetsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAssetsBlock();
  const result = await client.assets.tags.list();
}
```
