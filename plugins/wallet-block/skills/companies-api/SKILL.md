---
name: wallet-companies-api
description: Manage 23blocks Wallet Block companies via REST API. Use when creating companies, managing API keys, configuring exchange settings, impersonating users, or creating storage configurations.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Companies API

Complete API reference for 23blocks Wallet Block multi-tenant company management with API keys, exchange settings, user impersonation, and storage configuration.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://wallet.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Wallet API base URL | `https://wallet.api.us.23blocks.com` |
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

Retrieves company details by URL identifier.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/my-company" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url_id` | string | Yes | Company URL identifier (slug) |

**Response 200:**
```json
{
  "data": {
    "id": "company-uuid-123",
    "type": "company",
    "attributes": {
      "unique_id": "company-uuid-123",
      "url_id": "my-company",
      "name": "My Company Inc.",
      "status": "active",
      "settings": {
        "default_currency": "USD",
        "otp_enabled": true
      },
      "created_at": "2025-01-01T00:00:00Z",
      "updated_at": "2025-06-15T10:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Company not found

---

### POST /companies/ - Create Company

Creates a new company tenant.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "company": {
      "name": "My Company Inc.",
      "url_id": "my-company",
      "settings": {
        "default_currency": "USD",
        "otp_enabled": true
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Company display name |
| `url_id` | string | Yes | URL-safe company identifier (slug) |
| `settings` | object | No | Company-level settings |
| `settings.default_currency` | string | No | Default currency code |
| `settings.otp_enabled` | boolean | No | Enable OTP for transactions |

**Response 201:**
```json
{
  "data": {
    "id": "company-uuid-456",
    "type": "company",
    "attributes": {
      "unique_id": "company-uuid-456",
      "url_id": "my-company",
      "name": "My Company Inc.",
      "status": "active",
      "settings": {
        "default_currency": "USD",
        "otp_enabled": true
      },
      "created_at": "2025-06-20T10:00:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (duplicate url_id, missing name)

---

### GET /companies/:url_id/keys - List API Keys

Lists all API keys for a company.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/my-company/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url_id` | string | Yes | Company URL identifier |

**Response 200:**
```json
{
  "data": [
    {
      "id": "key-uuid-001",
      "type": "api_key",
      "attributes": {
        "unique_id": "key-uuid-001",
        "key": "pk_live_sh_f2b5ab3c7203d29b6d2937e2",
        "name": "Production Key",
        "environment": "live",
        "status": "active",
        "created_at": "2025-01-01T00:00:00Z"
      }
    },
    {
      "id": "key-uuid-002",
      "type": "api_key",
      "attributes": {
        "unique_id": "key-uuid-002",
        "key": "pk_test_sh_a1b2c3d4e5f6g7h8i9j0",
        "name": "Test Key",
        "environment": "test",
        "status": "active",
        "created_at": "2025-01-01T00:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 2
  }
}
```

---

### POST /companies/:unique_id/keys - Add API Key

Creates a new API key for a company.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/company-uuid-123/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key": {
      "name": "Mobile App Key",
      "environment": "live"
    }
  }'
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `unique_id` | string | Yes | Company unique identifier |

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Key display name |
| `environment` | enum | No | live, test (default: live) |

**Response 201:**
```json
{
  "data": {
    "id": "key-uuid-003",
    "type": "api_key",
    "attributes": {
      "unique_id": "key-uuid-003",
      "key": "pk_live_sh_new_key_value_here",
      "name": "Mobile App Key",
      "environment": "live",
      "status": "active",
      "created_at": "2025-06-20T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Company not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /companies/:unique_id/keys/:key_unique_id - Delete API Key

Deletes an API key from a company.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/companies/company-uuid-123/keys/key-uuid-003" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `unique_id` | string | Yes | Company unique identifier |
| `key_unique_id` | string | Yes | API key unique identifier |

**Response 204:** No content

**Errors:**
- `404 Not Found` - Company or key not found

---

### POST /companies/:unique_id/exchange - Add Exchange Settings

Configures exchange rate settings for a company. Used for multi-currency wallet operations.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/company-uuid-123/exchange" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "exchange": {
      "base_currency": "USD",
      "target_currency": "EUR",
      "rate": 0.92,
      "provider": "manual"
    }
  }'
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `unique_id` | string | Yes | Company unique identifier |

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `base_currency` | string | Yes | Source currency code |
| `target_currency` | string | Yes | Target currency code |
| `rate` | decimal | Yes | Exchange rate |
| `provider` | string | No | Rate provider (manual, api) |

**Response 201:**
```json
{
  "data": {
    "id": "exchange-uuid-001",
    "type": "exchange_setting",
    "attributes": {
      "unique_id": "exchange-uuid-001",
      "base_currency": "USD",
      "target_currency": "EUR",
      "rate": 0.92,
      "provider": "manual",
      "created_at": "2025-06-20T11:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Company not found
- `422 Unprocessable Entity` - Invalid currency pair or rate

---

### POST /companies/:url_id/access - Impersonate User

Generates an access token to perform operations on behalf of a company user. Used for administrative operations.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/my-company/access" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access": {
      "user_unique_id": "user-uuid-123",
      "reason": "Customer support - Wallet investigation"
    }
  }'
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url_id` | string | Yes | Company URL identifier |

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | string | Yes | Target user unique identifier |
| `reason` | string | No | Reason for impersonation (audit trail) |

**Response 200:**
```json
{
  "data": {
    "id": "access-uuid-001",
    "type": "access_token",
    "attributes": {
      "unique_id": "access-uuid-001",
      "user_unique_id": "user-uuid-123",
      "token": "imp_token_abc123def456",
      "expires_at": "2025-06-20T12:00:00Z",
      "created_at": "2025-06-20T11:00:00Z"
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Insufficient permissions for impersonation
- `404 Not Found` - Company or user not found

---

### POST /companies/:url_id/storage - Create Storage

Creates a storage configuration for company data.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/my-company/storage" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "storage": {
      "name": "Transaction Archives",
      "storage_type": "s3",
      "config": {
        "bucket": "my-company-wallet-archives",
        "region": "us-east-1"
      }
    }
  }'
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url_id` | string | Yes | Company URL identifier |

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Storage display name |
| `storage_type` | string | Yes | Storage type (e.g., s3, gcs, azure) |
| `config` | object | Yes | Provider-specific configuration |

**Response 201:**
```json
{
  "data": {
    "id": "storage-uuid-001",
    "type": "storage",
    "attributes": {
      "unique_id": "storage-uuid-001",
      "name": "Transaction Archives",
      "storage_type": "s3",
      "status": "active",
      "created_at": "2025-06-20T11:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Company not found
- `422 Unprocessable Entity` - Invalid storage configuration

---

## Data Models

### Company
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `url_id` | string | URL-safe company slug |
| `name` | string | Company display name |
| `status` | enum | active, inactive |
| `settings` | object | Company-level configuration |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### ApiKey
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `key` | string | API key value |
| `name` | string | Key display name |
| `environment` | enum | live, test |
| `status` | enum | active, revoked |
| `created_at` | timestamp | Creation time |

### ExchangeSetting
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `base_currency` | string | Source currency code |
| `target_currency` | string | Target currency code |
| `rate` | decimal | Exchange rate |
| `provider` | string | Rate provider |
| `created_at` | timestamp | Creation time |

### AccessToken
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | Impersonated user ID |
| `token` | string | Access token value |
| `expires_at` | timestamp | Token expiration time |
| `created_at` | timestamp | Creation time |

### Storage
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Storage display name |
| `storage_type` | string | Storage provider type |
| `status` | enum | active, inactive |
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
