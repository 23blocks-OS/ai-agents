---
name: geolocation-companies-api
description: Manage 23blocks geolocation companies (tenants) via REST API. Use when creating companies, managing API keys for external integrations, configuring RabbitMQ exchange settings, or generating impersonation tokens.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Companies API

Complete API reference for 23blocks multi-tenant company management, API keys, and exchange settings for the geolocation system.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://geolocation.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Geolocation API base URL | `https://geolocation.api.us.23blocks.com` |
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
      "created_at": "2025-01-10T10:30:00Z"
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
      "created_at": "2025-01-12T10:30:00Z"
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

### GET /companies/:url_id/keys - List Company API Keys

Lists all API keys for external integrations (e.g., Google Maps, Mapbox).

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/acme-corp/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "key-uuid-123",
      "type": "company_key",
      "attributes": {
        "unique_id": "key-uuid-123",
        "description": "Google Maps API Key",
        "provider": "google_maps",
        "api_region": "us-east-1",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /companies/:url_id/keys - Add Company API Key

Adds an API key for an external integration.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/acme-corp/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "company_key": {
      "description": "Google Maps API Key",
      "provider": "google_maps",
      "api_key": "AIzaSyB...",
      "api_secret": "",
      "api_region": "us-east-1"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `description` | string | Yes | Key description |
| `provider` | string | Yes | Provider name (google_maps, mapbox, etc.) |
| `api_key` | string | Yes | API key value |
| `api_secret` | string | No | API secret value |
| `api_region` | string | No | API region |

**Response 201:**
```json
{
  "data": {
    "id": "new-key-uuid",
    "type": "company_key",
    "attributes": {
      "unique_id": "new-key-uuid",
      "description": "Google Maps API Key",
      "provider": "google_maps",
      "api_region": "us-east-1",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /companies/:url_id/keys/:key_unique_id - Delete Company API Key

Deletes a company API key.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/companies/acme-corp/keys/key-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Exchange Settings

### POST /companies/:url_id/exchange - Add Exchange Settings

Configures RabbitMQ exchange settings for async messaging.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/acme-corp/exchange" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "exchange": {
      "host": "rabbitmq.example.com",
      "port": 5672,
      "user_name": "geolocation_user",
      "password": "secure_password",
      "vhost": "/acme"
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
      "vhost": "/acme",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Storage Settings

### POST /companies/:url_id/storage - Configure Storage

Configures cloud storage settings for the company (images, files).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/acme-corp/storage" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "storage": {
      "provider": "aws_s3",
      "bucket": "acme-geolocation-assets",
      "region": "us-east-1",
      "access_key": "AKIAIOSFODNN7EXAMPLE",
      "secret_key": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `provider` | string | Yes | Storage provider (aws_s3, gcs, azure_blob) |
| `bucket` | string | Yes | Storage bucket name |
| `region` | string | No | Storage region |
| `access_key` | string | Yes | Storage access key |
| `secret_key` | string | Yes | Storage secret key |

**Response 201:**
```json
{
  "data": {
    "id": "storage-uuid-123",
    "type": "storage_settings",
    "attributes": {
      "unique_id": "storage-uuid-123",
      "provider": "aws_s3",
      "bucket": "acme-geolocation-assets",
      "region": "us-east-1",
      "created_at": "2025-01-12T10:30:00Z"
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

### StorageSettings
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `provider` | string | Storage provider |
| `bucket` | string | Bucket name |
| `region` | string | Storage region |

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
