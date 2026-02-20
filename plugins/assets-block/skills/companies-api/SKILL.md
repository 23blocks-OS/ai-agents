---
name: assets-companies-api
description: Manage 23blocks multi-tenant companies via REST API. Use when retrieving company info, managing API keys, exchanging keys, impersonating users, or checking storage within the Assets Block.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Companies API

Complete API reference for 23blocks Assets Block multi-tenant company management with API key CRUD, exchange, impersonation, and storage.

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
curl -X GET "$BLOCKS_API_URL/companies/my-company" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /companies/:url_id - Get Company

Retrieves company information by URL identifier.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/my-company" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "company-uuid-123",
    "type": "company",
    "attributes": {
      "unique_id": "company-uuid-123",
      "name": "Acme Corporation",
      "url_id": "my-company",
      "description": "Enterprise asset management",
      "plan": "enterprise",
      "status": "active",
      "assets_count": 1500,
      "users_count": 250,
      "storage_used_mb": 4500,
      "storage_limit_mb": 10000,
      "created_at": "2024-06-01T10:00:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Company not found

---

### POST /companies/ - Create Company

Creates a new company (tenant).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "company": {
      "name": "New Company Inc.",
      "url_id": "new-company",
      "description": "New tenant for asset management"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Company name |
| `url_id` | string | Yes | URL-safe identifier (unique) |
| `description` | string | No | Company description |

**Response 201:**
```json
{
  "data": {
    "id": "new-company-uuid",
    "type": "company",
    "attributes": {
      "unique_id": "new-company-uuid",
      "name": "New Company Inc.",
      "url_id": "new-company",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (e.g., url_id taken)

---

## API Keys

### GET /companies/:url_id/keys - List Keys

Lists all API keys for the company.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/my-company/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "key-uuid-001",
      "type": "api_key",
      "attributes": {
        "unique_id": "key-uuid-001",
        "name": "Production Key",
        "key_prefix": "pk_live_sh_f2b5",
        "key_type": "live",
        "status": "active",
        "last_used_at": "2025-01-12T10:30:00Z",
        "created_at": "2024-06-01T10:00:00Z"
      }
    },
    {
      "id": "key-uuid-002",
      "type": "api_key",
      "attributes": {
        "unique_id": "key-uuid-002",
        "name": "Test Key",
        "key_prefix": "pk_test_sh_a1b2",
        "key_type": "test",
        "status": "active",
        "last_used_at": "2025-01-11T09:00:00Z",
        "created_at": "2024-06-01T10:00:00Z"
      }
    }
  ]
}
```

---

### POST /companies/:url_id/keys - Create Key

Creates a new API key for the company.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/my-company/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key": {
      "name": "Staging Key",
      "key_type": "test"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Key name/label |
| `key_type` | enum | No | live, test (default: test) |

**Response 201:**
```json
{
  "data": {
    "id": "new-key-uuid",
    "type": "api_key",
    "attributes": {
      "unique_id": "new-key-uuid",
      "name": "Staging Key",
      "key": "pk_test_sh_c3d4e5f6g7h8i9j0",
      "key_type": "test",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Note:** The full key value is only returned once at creation time. Store it securely.

---

### DELETE /companies/:url_id/keys/:key_id - Delete Key

Deletes an API key.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/companies/my-company/keys/key-uuid-002" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Key not found

---

### POST /companies/:url_id/keys/exchange - Exchange Key

Exchanges one API key for another (key rotation).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/my-company/keys/exchange" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "exchange": {
      "current_key": "pk_live_sh_f2b5ab3c7203d29b6d2937e2",
      "name": "Rotated Production Key"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `current_key` | string | Yes | The current key to exchange |
| `name` | string | No | Name for the new key |

**Response 200:**
```json
{
  "data": {
    "id": "new-key-uuid",
    "type": "api_key",
    "attributes": {
      "unique_id": "new-key-uuid",
      "name": "Rotated Production Key",
      "key": "pk_live_sh_new_key_value_here",
      "key_type": "live",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Impersonation

### POST /companies/:url_id/impersonate - Impersonate User

Generates an impersonation token for a user within the company.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/my-company/impersonate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "impersonate": {
      "user_unique_id": "user-uuid-789"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User to impersonate |

**Response 200:**
```json
{
  "data": {
    "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.impersonation_token...",
    "user_unique_id": "user-uuid-789",
    "user_name": "Target User",
    "expires_at": "2025-01-12T11:30:00Z"
  }
}
```

**Errors:**
- `403 Forbidden` - Insufficient permissions for impersonation
- `404 Not Found` - User not found in company

---

## Storage

### GET /companies/:url_id/storage - Get Storage Info

Retrieves storage usage information for the company.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/my-company/storage" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "storage_info",
    "attributes": {
      "used_mb": 4500,
      "limit_mb": 10000,
      "usage_percentage": 45.0,
      "files_count": 2340,
      "breakdown": {
        "asset_images_mb": 3200,
        "event_images_mb": 850,
        "category_images_mb": 150,
        "other_mb": 300
      }
    }
  }
}
```

---

## Data Models

### Company
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Company name |
| `url_id` | string | URL-safe identifier |
| `description` | string | Company description |
| `plan` | string | Subscription plan |
| `status` | enum | active, suspended, cancelled |
| `assets_count` | integer | Total assets |
| `users_count` | integer | Total users |
| `storage_used_mb` | integer | Storage used in MB |
| `storage_limit_mb` | integer | Storage limit in MB |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### ApiKey
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Key name/label |
| `key` | string | Full key (only at creation) |
| `key_prefix` | string | Key prefix for identification |
| `key_type` | enum | live, test |
| `status` | enum | active, revoked |
| `last_used_at` | timestamp | Last usage time |
| `created_at` | timestamp | Creation time |

### StorageInfo
| Field | Type | Description |
|-------|------|-------------|
| `used_mb` | integer | Storage used in MB |
| `limit_mb` | integer | Storage limit in MB |
| `usage_percentage` | float | Usage percentage |
| `files_count` | integer | Total number of files |
| `breakdown` | object | Storage breakdown by type |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "403",
    "code": "forbidden",
    "title": "Access Denied",
    "detail": "You do not have permission to impersonate users."
  }]
}
```

| Code | Description |
|------|-------------|
| `401` | Unauthorized - Invalid or missing credentials |
| `403` | Forbidden - Insufficient permissions |
| `404` | Not Found - Company, key, or user not found |
| `422` | Unprocessable Entity - Validation errors |
