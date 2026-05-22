---
name: 23blocks-company-companies-api
description: Manage 23blocks company tenants via REST API. Use when creating companies, managing API keys for external integrations, configuring RabbitMQ exchange settings, or checking service health.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.1"
  verified-by: 23blocks-api-company
  verified-date: "2026-05-18"
---

# Companies API

Complete API reference for 23blocks multi-tenant company management, API keys, and exchange settings.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Company API base URL | `https://company.api.us.23blocks.com` |
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
export BLOCKS_API_URL="https://company.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://company.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### GET /companies/:url_id - Get Company

Retrieves company details by URL identifier.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/acme-corp" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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

Creates a new company (tenant). Requires scope: `tenants:create`.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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

Lists all API keys for external integrations (e.g., OpenAI, AWS). Requires scope: `keys:read`.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/acme-corp/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
        "description": "OpenAI API Key",
        "provider": "openai",
        "api_region": "us-east-1",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /companies/:url_id/keys - Add Company API Key

Adds an API key for an external integration. Requires scope: `keys:write`.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/acme-corp/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "company_key": {
      "description": "AWS CloudSearch Key",
      "provider": "aws",
      "api_key": "AKIAIOSFODNN7EXAMPLE",
      "api_secret": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
      "api_region": "us-east-1"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `description` | string | Yes | Key description |
| `provider` | string | Yes | Provider name (openai, aws, etc.) |
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
      "description": "AWS CloudSearch Key",
      "provider": "aws",
      "api_region": "us-east-1",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /companies/:url_id/keys/:key_unique_id - Delete Company API Key

Deletes a company API key. Requires scope: `keys:write`.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/companies/acme-corp/keys/key-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** Empty body `{}`

---

## Exchange Settings

### POST /companies/:url_id/exchange - Add Exchange Settings

Configures RabbitMQ exchange settings for async messaging. Requires scope: `exchange:write`.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/acme-corp/exchange" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "exchange": {
      "host": "rabbitmq.example.com",
      "port": 5672,
      "user_name": "company_user",
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

## Health Endpoints

### GET /health - Basic Health Check

```bash
curl -X GET "$BLOCKS_API_URL/health"
```

### GET /health/ready - Readiness Check

```bash
curl -X GET "$BLOCKS_API_URL/health/ready"
```

### GET /health/detailed - Detailed Health Status

```bash
curl -X GET "$BLOCKS_API_URL/health/detailed"
```

---

## Auth Gates

All `/companies/*` endpoints require a valid JWT + valid X-API-Key. Some sub-actions require additional scopes:

| Endpoint | Required Scope |
|----------|---------------|
| `POST /companies` | `tenants:create` |
| `GET /companies/:url_id/keys` | `keys:read` |
| `POST /companies/:url_id/keys` | `keys:write` |
| `DELETE /companies/:url_id/keys/:key_unique_id` | `keys:write` |
| `POST /companies/:url_id/exchange` | `exchange:write` |

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
    "source": { "pointer": "/companies" },
    "code": "forbidden",
    "title": "Provisioning Not Authorized",
    "detail": "You are not authorized to create companies."
  }]
}
```
