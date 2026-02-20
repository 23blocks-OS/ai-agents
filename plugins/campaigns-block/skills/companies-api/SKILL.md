---
name: campaigns-companies-api
description: Manage 23blocks campaigns companies (tenants) via REST API. Use when creating companies, managing API keys for external integrations, configuring RabbitMQ exchange settings, or generating impersonation tokens.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Companies API

Complete API reference for 23blocks multi-tenant company management, API keys, and exchange settings for the Campaigns Block.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://campaigns.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Campaigns API base URL | `https://campaigns.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/companies/acme-corp" \
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
curl -X GET "$BLOCKS_API_URL/companies/acme-corp" \
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
      "code": "ACME",
      "name": "Acme Corporation",
      "url_id": "acme-corp",
      "schema_name": "acme_corp",
      "auth_provider": "23blocks",
      "preferred_domain": "acme.example.com",
      "preferred_language": "en",
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Company not found

---

### POST /companies - Create Company

Creates a new company (tenant). Requires provisioning authorization.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "company": {
      "code": "NEWCO",
      "name": "New Company Inc",
      "url_id": "new-company",
      "auth_provider": "23blocks",
      "preferred_language": "en"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Company code (uppercase) |
| `name` | string | Yes | Company display name |
| `url_id` | string | Yes | URL-friendly identifier |
| `auth_provider` | string | No | Auth provider (23blocks, auth0, cognito, okta) |
| `preferred_domain` | string | No | Preferred domain |
| `preferred_language` | string | No | Language code (default: en) |

**Response 201:**
```json
{
  "data": {
    "id": "new-company-uuid",
    "type": "company",
    "attributes": {
      "unique_id": "new-company-uuid",
      "code": "NEWCO",
      "name": "New Company Inc",
      "url_id": "new-company",
      "schema_name": "new_company",
      "api_access_key": "generated-access-key",
      "auth_provider": "23blocks",
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Provisioning not authorized
- `409 Conflict` - Company code or url_id already exists
- `422 Unprocessable Entity` - Validation errors

---

## Company API Keys

### POST /companies/:unique_id/keys - Add Company API Key

Adds an API key for an external integration.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/company-uuid-123/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "company_key": {
      "description": "Facebook Ads API Key",
      "provider": "facebook",
      "api_key": "EAABsbCS1ZB...",
      "api_secret": "a1b2c3d4e5f6..."
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `description` | string | Yes | Key description |
| `provider` | string | Yes | Provider name (facebook, google, aws, etc.) |
| `api_key` | string | Yes | API key value |
| `api_secret` | string | No | API secret value |
| `api_region` | string | No | API region |

**Response 201:**
```json
{
  "data": {
    "id": "key-uuid-123",
    "type": "company_key",
    "attributes": {
      "unique_id": "key-uuid-123",
      "description": "Facebook Ads API Key",
      "provider": "facebook",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Company not found
- `422 Unprocessable Entity` - Validation errors

---

## Exchange Settings

### POST /companies/:unique_id/exchange - Add Exchange Settings

Configures RabbitMQ exchange settings for async messaging.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/company-uuid-123/exchange" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "exchange": {
      "host": "rabbitmq.example.com",
      "port": 5672,
      "user_name": "campaigns_user",
      "password": "secure_password",
      "vhost": "/campaigns"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `host` | string | Yes | RabbitMQ host |
| `port` | integer | No | Port (default: 5672) |
| `user_name` | string | Yes | Connection username |
| `password` | string | Yes | Connection password |
| `vhost` | string | No | Virtual host |

**Response 201:**
```json
{
  "data": {
    "id": "exchange-uuid-123",
    "type": "exchange_settings",
    "attributes": {
      "unique_id": "exchange-uuid-123",
      "host": "rabbitmq.example.com",
      "port": 5672,
      "vhost": "/campaigns",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

## Impersonation

### POST /companies/:url_id/access - Impersonate User

Generates a temporary access token for a specified user. Used for administrative and debugging purposes.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/acme-corp/access" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "user-uuid-456",
    "elevated": false
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User to impersonate |
| `elevated` | boolean | No | Grant elevated permissions |

**Response 200:**
```json
{
  "data": {
    "access_token": "temporary-jwt-token...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "user_unique_id": "user-uuid-456"
  }
}
```

**Errors:**
- `403 Forbidden` - Not authorized to impersonate
- `404 Not Found` - User not found

---

## Data Models

### Company
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `code` | string | Company code (uppercase) |
| `name` | string | Display name |
| `url_id` | string | URL-friendly identifier |
| `schema_name` | string | Database schema name |
| `api_access_key` | string | Company API access key |
| `auth_provider` | string | Auth provider (23blocks, auth0, cognito, okta) |
| `preferred_domain` | string | Preferred domain |
| `preferred_language` | string | Language code |
| `status` | string | active, inactive |
| `created_at` | timestamp | Creation time |

### CompanyKey
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `description` | string | Key description |
| `provider` | string | Provider name |
| `api_key` | string | API key (write-only) |
| `api_secret` | string | API secret (write-only) |
| `api_region` | string | API region |

### ExchangeSettings
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `host` | string | RabbitMQ host |
| `port` | integer | Connection port |
| `user_name` | string | Connection username |
| `vhost` | string | Virtual host |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "403",
    "code": "forbidden",
    "title": "Provisioning Not Authorized",
    "detail": "You are not authorized to create companies."
  }]
}
```
