---
name: crm-companies-api
description: Manage 23blocks CRM companies via REST API. Use when configuring multi-tenant company settings, managing API keys, performing key exchanges, impersonating users, and managing storage.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# CRM Companies API

Complete API reference for 23blocks CRM multi-tenant company configuration with API key management, key exchange, user impersonation, and storage.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://crm.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
echo "All credentials configured"
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | CRM API base URL | `https://crm.api.us.23blocks.com` |
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

## Company Endpoints

### GET /companies/:url_id - Get Company

Retrieves company configuration by URL identifier.

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
      "name": "Acme Corp",
      "url_id": "my-company",
      "domain": "acme.com",
      "plan": "professional",
      "status": "active",
      "settings": {
        "timezone": "America/New_York",
        "locale": "en-US",
        "currency": "USD"
      },
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Company not found

---

### POST /companies/ - Create Company

Creates a new company in the multi-tenant system.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "company": {
      "name": "New Company",
      "url_id": "new-company",
      "domain": "newcompany.com",
      "settings": {
        "timezone": "America/Chicago",
        "locale": "en-US",
        "currency": "USD"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Company name |
| `url_id` | string | Yes | URL-safe identifier |
| `domain` | string | No | Company domain |
| `settings` | object | No | Company settings (timezone, locale, currency) |

**Response 201:**
```json
{
  "data": {
    "id": "company-uuid-456",
    "type": "company",
    "attributes": {
      "unique_id": "company-uuid-456",
      "name": "New Company",
      "url_id": "new-company",
      "domain": "newcompany.com",
      "status": "active",
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

## API Key Management

### GET /companies/:url_id/keys - List API Keys

Lists all API keys for a company.

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
      "id": "key-uuid-123",
      "type": "api_key",
      "attributes": {
        "unique_id": "key-uuid-123",
        "name": "Production Key",
        "key_prefix": "pk_live_sh_f2b5ab3c",
        "environment": "live",
        "status": "active",
        "last_used_at": "2026-02-15T10:00:00Z",
        "created_at": "2026-01-01T00:00:00Z"
      }
    }
  ]
}
```

---

### POST /companies/:url_id/keys - Create API Key

Creates a new API key for a company.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/my-company/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": {
      "name": "Staging Key",
      "environment": "test"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Key name |
| `environment` | string | No | live, test (default: test) |

**Response 201:**
```json
{
  "data": {
    "id": "key-uuid-456",
    "type": "api_key",
    "attributes": {
      "unique_id": "key-uuid-456",
      "name": "Staging Key",
      "key": "pk_test_sh_a1b2c3d4e5f6g7h8",
      "environment": "test",
      "status": "active",
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

> **Note:** The full API key value is only returned once at creation time. Store it securely.

---

### PUT /companies/:url_id/keys/:key_id - Update API Key

Updates an existing API key.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/companies/my-company/keys/key-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": {
      "name": "Production Key - Primary",
      "status": "active"
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
      "name": "Production Key - Primary",
      "status": "active",
      "updated_at": "2026-02-15T14:00:00Z"
    }
  }
}
```

---

### DELETE /companies/:url_id/keys/:key_id - Delete API Key

Deletes an API key.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/companies/my-company/keys/key-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Key Exchange

### POST /companies/:url_id/exchange - Exchange Key

Exchanges a secret key for an authentication token.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/my-company/exchange" \
  -H "Content-Type: application/json" \
  -d '{
    "secret_key": "sk_live_sh_x1y2z3a4b5c6d7e8"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `secret_key` | string | Yes | Secret key to exchange |

**Response 200:**
```json
{
  "data": {
    "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_at": "2026-02-16T10:30:00Z"
  }
}
```

**Errors:**
- `401 Unauthorized` - Invalid secret key

---

## User Impersonation

### POST /companies/:url_id/impersonate - Impersonate User

Generates a token to impersonate a user within the company.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/my-company/impersonate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user-uuid-123"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | uuid | Yes | User ID to impersonate |

**Response 200:**
```json
{
  "data": {
    "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
    "impersonated_user_id": "user-uuid-123",
    "expires_at": "2026-02-15T11:30:00Z"
  }
}
```

**Errors:**
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - User not found

---

## Storage

### GET /companies/:url_id/storage - Get Storage Info

Retrieves storage usage information for a company.

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
    "company_id": "company-uuid-123",
    "storage_used_bytes": 2147483648,
    "storage_used_display": "2.0 GB",
    "storage_limit_bytes": 10737418240,
    "storage_limit_display": "10.0 GB",
    "usage_percentage": 20.0,
    "documents_count": 156,
    "logos_count": 12,
    "breakdown": {
      "documents": 1800000000,
      "logos": 50000000,
      "other": 297483648
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
| `domain` | string | Company domain |
| `plan` | string | Subscription plan |
| `status` | enum | active, inactive, suspended |
| `settings` | object | Company settings |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### APIKey
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Key name |
| `key_prefix` | string | First 8 chars of the key |
| `environment` | enum | live, test |
| `status` | enum | active, revoked |
| `last_used_at` | timestamp | Last usage time |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Company Not Found",
    "detail": "The requested company could not be found."
  }]
}
```
