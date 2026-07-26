---
name: 23blocks-search-identities-api
description: Manage 23blocks Search user identities via REST API. Use when registering users in the search block, retrieving identity details, or updating user profiles.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks Search identity registration and management.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Search API base URL | `https://search.api.us.23blocks.com` |
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
export BLOCKS_API_URL="https://search.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://search.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### GET /identities/:unique_id - Get Identity

Retrieves a user identity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "user_unique_id": "user-uuid-123",
      "name": "John Doe",
      "email": "user@example.com",
      "phone": "+1234567890",
      "avatar_url": "https://example.com/avatar.jpg",
      "role_id": 4,
      "role_name": "user",
      "status": "active",
      "enabled": true,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Identity not found

---

### POST /identities/:unique_id/register - Register Identity

Registers a new user identity in the search block. The `:unique_id` in the path is the user's `user_unique_id` from the Auth (Gateway) block and is the only required value.

> **Note:** Block identity records are notification routing caches, not identity models. The canonical user record lives in the Auth (Gateway) block. `email`/`phone` here are optional denormalized routing fields; duplicates across users are allowed.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/user-uuid-123/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "name": "New User",
      "phone": "+1234567890"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | No | Optional denormalized routing field; if blank, the block skips email notifications |
| `name` | string | No | User display name |
| `phone` | string | No | Optional denormalized routing field; if blank, the block skips SMS notifications |

**Response 201:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "user_unique_id": "user-uuid-123",
      "email": "newuser@example.com",
      "name": "New User",
      "role_id": 4,
      "role_name": "user",
      "status": "active",
      "enabled": true,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Duplicate registration of the same `user_unique_id`
- `422 Unprocessable Entity` - Missing `user_unique_id`

---

### PUT /identities/:unique_id - Update Identity

Updates an existing user identity.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/identities/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "name": "John Updated",
      "avatar_url": "https://example.com/new-avatar.jpg"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "name": "John Updated",
      "avatar_url": "https://example.com/new-avatar.jpg",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Data Models

### UserIdentity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | User unique ID from auth provider |
| `name` | string | Display name |
| `email` | string | Email address |
| `phone` | string | Phone number |
| `avatar_url` | string | Avatar image URL |
| `role_id` | integer | Role ID (1=admin, 2=manager, 4=user) |
| `role_name` | string | Role name |
| `status` | string | Account status (active, inactive) |
| `enabled` | boolean | Whether user is enabled |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Identity Not Found",
    "detail": "The requested identity could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-search
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
// IdentitiesService — client.search.identities
client.search.identities.list(params?: ListIdentitiesParams): Promise<PageResult<SearchIdentity>>;
client.search.identities.get(uniqueId: string): Promise<SearchIdentity>;
client.search.identities.register(uniqueId: string, data: RegisterIdentityRequest): Promise<SearchIdentity>;
client.search.identities.update(uniqueId: string, data: UpdateIdentityRequest): Promise<SearchIdentity>;
```

### TypeScript Types

```typescript
import type {
  SearchIdentity,
  RegisterIdentityRequest,
  UpdateIdentityRequest,
  ListIdentitiesParams,
} from '@23blocks/block-search';
```

### React Hook

```typescript
import { useSearchBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useSearchBlock();
  const result = await client.search.identities.list({ page: 1, perPage: 20 });
}
```
