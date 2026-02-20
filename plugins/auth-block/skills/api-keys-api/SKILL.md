---
name: auth-api-keys-api
description: Manage 23blocks API keys via REST API. Use when creating, updating, rotating, revoking API keys, managing key scopes, or viewing key usage statistics.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# API Keys API

Complete API reference for 23blocks API key lifecycle management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://auth.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Auth API base URL | `https://auth.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/api_keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /api_keys - List API Keys

Lists all API keys for the current account.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/api_keys?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "key-uuid-123",
      "type": "api_key",
      "attributes": {
        "unique_id": "key-uuid-123",
        "name": "Production Key",
        "prefix": "pk_live_sh",
        "scopes": ["read", "write"],
        "status": "active",
        "last_used_at": "2025-01-12T10:30:00Z",
        "expires_at": "2026-01-12T10:30:00Z",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 3
  }
}
```

---

### GET /api_keys/:unique_id - Get API Key

Retrieves a single API key by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/api_keys/key-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "key-uuid-123",
    "type": "api_key",
    "attributes": {
      "unique_id": "key-uuid-123",
      "name": "Production Key",
      "prefix": "pk_live_sh",
      "scopes": ["read", "write"],
      "status": "active",
      "last_used_at": "2025-01-12T10:30:00Z",
      "expires_at": "2026-01-12T10:30:00Z",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - API key not found

---

### POST /api_keys - Create API Key

Creates a new API key.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/api_keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": {
      "name": "Staging Key",
      "scopes": ["read"],
      "expires_at": "2026-06-01T00:00:00Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Key display name |
| `scopes` | array | No | Permission scopes |
| `expires_at` | timestamp | No | Expiration date |

**Response 201:**
```json
{
  "data": {
    "id": "new-key-uuid",
    "type": "api_key",
    "attributes": {
      "unique_id": "new-key-uuid",
      "name": "Staging Key",
      "key": "pk_live_sh_a1b2c3d4e5f6g7h8",
      "scopes": ["read"],
      "status": "active",
      "expires_at": "2026-06-01T00:00:00Z",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

> **Note:** The full `key` value is only returned on creation. Store it securely.

---

### PUT /api_keys/:unique_id - Update API Key

Updates an existing API key.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/api_keys/key-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": {
      "name": "Updated Key Name"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "key-uuid-123",
    "type": "api_key",
    "attributes": {
      "unique_id": "key-uuid-123",
      "name": "Updated Key Name",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /api_keys/:unique_id - Delete API Key

Permanently deletes an API key.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/api_keys/key-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### POST /api_keys/:unique_id/rotate - Rotate API Key

Rotates an API key, generating a new secret while keeping the same key ID.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/api_keys/key-uuid-123/rotate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "key-uuid-123",
    "type": "api_key",
    "attributes": {
      "unique_id": "key-uuid-123",
      "key": "pk_live_sh_new_secret_value",
      "rotated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

> **Note:** The previous key becomes invalid immediately after rotation.

---

### PUT /api_keys/:unique_id/scopes - Update Key Scopes

Updates the permission scopes for an API key.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/api_keys/key-uuid-123/scopes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "scopes": ["read", "write", "admin"]
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "key-uuid-123",
    "type": "api_key",
    "attributes": {
      "unique_id": "key-uuid-123",
      "scopes": ["read", "write", "admin"],
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### POST /api_keys/:unique_id/revoke - Revoke API Key

Immediately revokes an API key, preventing any further use.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/api_keys/key-uuid-123/revoke" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "key-uuid-123",
    "type": "api_key",
    "attributes": {
      "unique_id": "key-uuid-123",
      "status": "revoked",
      "revoked_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### GET /api_keys/:unique_id/usage - Get Key Usage Stats

Retrieves usage statistics for an API key.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/api_keys/key-uuid-123/usage" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "key-uuid-123",
    "type": "api_key_usage",
    "attributes": {
      "total_requests": 15420,
      "requests_today": 342,
      "requests_this_month": 8500,
      "last_used_at": "2025-01-12T10:30:00Z",
      "top_endpoints": [
        { "endpoint": "GET /users", "count": 5200 },
        { "endpoint": "POST /sessions", "count": 3100 }
      ]
    }
  }
}
```

---

## Data Models

### ApiKey
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Key display name |
| `key` | string | Full key (only on create/rotate) |
| `prefix` | string | Key prefix for identification |
| `scopes` | array | Permission scopes |
| `status` | enum | active, revoked, expired |
| `last_used_at` | timestamp | Last usage time |
| `expires_at` | timestamp | Expiration date |
| `revoked_at` | timestamp | Revocation time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "API Key Not Found",
    "detail": "The requested API key could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-authentication
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
// ApiKeysService — client.authentication.apiKeys
list(params?: ListParams): Promise<PageResult<ApiKey>>;
get(uniqueId: string): Promise<ApiKey>;
getByKeyId(keyId: string): Promise<ApiKey>;
create(request: CreateApiKeyRequest): Promise<ApiKeyWithSecret>;
update(uniqueId: string, request: UpdateApiKeyRequest): Promise<ApiKey>;
regenerate(uniqueId: string): Promise<ApiKeyWithSecret>;
revoke(uniqueId: string, request?: RevokeApiKeyRequest): Promise<ApiKey>;
delete(uniqueId: string): Promise<void>;
getUsage(uniqueId: string, period?: 'day' | 'week' | 'month'): Promise<ApiKeyUsageStats>;
```

### TypeScript Types

```typescript
import type {
  ApiKey,
  ApiKeyWithSecret,
  CreateApiKeyRequest,
  UpdateApiKeyRequest,
  RevokeApiKeyRequest,
} from '@23blocks/block-authentication';

// Also exported from the service file:
import type {
  ApiKeyUsageStats,
} from '@23blocks/block-authentication';
```

### React Hook

```typescript
import { useAuthenticationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAuthenticationBlock();

  // Example: List all API keys
  const result = await client.authentication.apiKeys.list();
}
```
