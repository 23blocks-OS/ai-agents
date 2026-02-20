---
name: sales-companies-api
description: Manage 23blocks Sales multi-tenant companies via REST API. Use when creating companies, managing API keys, exchanging tokens between companies, impersonating users, or checking storage usage.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Companies API

Complete API reference for 23blocks Sales multi-tenant company management including API keys, token exchange, impersonation, and storage.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://sales.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Sales API base URL | `https://sales.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/companies" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Company CRUD

### GET /companies - List Companies

Lists all companies in the account.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies?page=1&records=20" \
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
      "id": "company-uuid-123",
      "type": "company",
      "attributes": {
        "unique_id": "company-uuid-123",
        "name": "Acme Corp",
        "slug": "acme-corp",
        "domain": "acme.com",
        "status": "active",
        "plan": "enterprise",
        "users_count": 45,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 15
  }
}
```

---

### GET /companies/:unique_id - Get Company

Retrieves a single company.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/company-uuid-123" \
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
      "slug": "acme-corp",
      "domain": "acme.com",
      "status": "active",
      "plan": "enterprise",
      "users_count": 45,
      "settings": {
        "default_currency": "USD",
        "timezone": "America/New_York",
        "billing_email": "billing@acme.com"
      },
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Company not found

---

### POST /companies - Create Company

Creates a new company.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "company": {
      "name": "New Company",
      "slug": "new-company",
      "domain": "newcompany.com",
      "settings": {
        "default_currency": "USD",
        "timezone": "America/New_York"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Company name |
| `slug` | string | No | URL-safe identifier |
| `domain` | string | No | Company domain |
| `settings` | object | No | Company settings |

**Response 201:**
```json
{
  "data": {
    "id": "company-uuid-new",
    "type": "company",
    "attributes": {
      "unique_id": "company-uuid-new",
      "name": "New Company",
      "slug": "new-company",
      "domain": "newcompany.com",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /companies/:unique_id - Update Company

Updates an existing company.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/companies/company-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "company": {
      "name": "Acme Corporation",
      "settings": {
        "default_currency": "EUR",
        "billing_email": "finance@acme.com"
      }
    }
  }'
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
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## API Keys

### GET /companies/:unique_id/keys - List API Keys

Lists all API keys for a company.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/company-uuid-123/keys" \
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
        "environment": "live",
        "status": "active",
        "last_used_at": "2025-01-12T09:00:00Z",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /companies/:unique_id/keys - Create API Key

Creates a new API key for a company.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/company-uuid-123/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key": {
      "name": "Staging Key",
      "environment": "test"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Key display name |
| `environment` | enum | No | live, test (default: live) |

**Response 201:**
```json
{
  "data": {
    "id": "key-uuid-new",
    "type": "api_key",
    "attributes": {
      "unique_id": "key-uuid-new",
      "name": "Staging Key",
      "key": "pk_test_sh_a1b2c3d4e5f6g7h8i9j0",
      "environment": "test",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Note:** The full `key` value is only returned once at creation time. Store it securely.

---

### DELETE /companies/:unique_id/keys/:key_id - Delete API Key

Deletes an API key.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/companies/company-uuid-123/keys/key-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Token Exchange

### POST /companies/:unique_id/exchange - Exchange Token

Exchanges a token to operate in the context of a different company.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/company-uuid-456/exchange" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response 200:**
```json
{
  "data": {
    "type": "token_exchange",
    "attributes": {
      "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.exchanged...",
      "company_id": "company-uuid-456",
      "company_name": "Partner Corp",
      "expires_at": "2025-01-12T12:30:00Z"
    }
  }
}
```

---

## Impersonation

### POST /companies/:unique_id/impersonate - Impersonate User

Generates a token to act as a specific user within a company.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/company-uuid-123/impersonate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "impersonate": {
      "user_id": "user-uuid-456"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | uuid | Yes | User to impersonate |

**Response 200:**
```json
{
  "data": {
    "type": "impersonation",
    "attributes": {
      "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.impersonated...",
      "user_id": "user-uuid-456",
      "user_email": "user@example.com",
      "company_id": "company-uuid-123",
      "expires_at": "2025-01-12T12:30:00Z"
    }
  }
}
```

---

## Storage

### GET /companies/:unique_id/storage - Get Storage Usage

Retrieves storage usage information for a company.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/company-uuid-123/storage" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "storage",
    "attributes": {
      "company_id": "company-uuid-123",
      "used_bytes": 5368709120,
      "used_gb": 5.0,
      "limit_bytes": 53687091200,
      "limit_gb": 50.0,
      "usage_percentage": 10.0,
      "breakdown": {
        "documents": { "count": 120, "bytes": 2147483648 },
        "images": { "count": 450, "bytes": 1073741824 },
        "exports": { "count": 30, "bytes": 536870912 },
        "other": { "count": 15, "bytes": 1610612736 }
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
| `slug` | string | URL-safe identifier |
| `domain` | string | Company domain |
| `status` | enum | active, inactive, suspended |
| `plan` | string | Subscription plan |
| `users_count` | integer | Number of users |
| `settings` | object | Company settings |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### APIKey
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Key display name |
| `key` | string | Full API key (only at creation) |
| `key_prefix` | string | Key prefix for identification |
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
