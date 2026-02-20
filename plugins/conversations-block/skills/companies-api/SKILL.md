---
name: conversations-companies-api
description: Multi-tenant company management, API keys, key exchange, and impersonation in the 23blocks Conversations Block
version: 1.0.0
tags:
  - conversations
  - companies
  - multi-tenant
  - api-keys
  - impersonation
env:
  - BLOCKS_API_URL
  - BLOCKS_AUTH_TOKEN
  - BLOCKS_API_KEY
---

# Companies API

Multi-tenant company management for the Conversations Block. Manage company profiles, API keys, key exchange for cross-block communication, and impersonation for administrative operations.

## Authentication

All requests require the following headers:

```
Authorization: Bearer $BLOCKS_AUTH_TOKEN
AppId: $BLOCKS_API_KEY
```

## Base URL

```
https://conversations.api.us.23blocks.com
```

---

## Endpoints

### Get Company

Retrieve a company profile by its URL identifier.

**Endpoint:** `GET /companies/:url_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| url_id | string | Yes | The company's URL-friendly identifier |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/companies/acme-corp" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "comp_001",
    "type": "companies",
    "attributes": {
      "unique_id": "comp_001",
      "name": "Acme Corp",
      "url_id": "acme-corp",
      "domain": "acme.com",
      "logo_url": "https://cdn.23blocks.com/companies/acme-corp/logo.png",
      "settings": {
        "max_group_size": 100,
        "max_file_size_mb": 50,
        "enable_meetings": true,
        "enable_push_notifications": true
      },
      "plan": "pro",
      "status": "active",
      "created_at": "2024-06-01T10:00:00Z",
      "updated_at": "2025-01-10T15:00:00Z"
    }
  }
}
```

---

### Create Company

Register a new company in the Conversations Block.

**Endpoint:** `POST /companies/`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Company name |
| url_id | string | Yes | URL-friendly identifier (unique) |
| domain | string | No | Company domain |
| logo_url | string | No | URL to company logo |
| settings | object | No | Company-specific settings |
| metadata | object | No | Additional metadata |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/companies/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Acme Corp",
    "url_id": "acme-corp",
    "domain": "acme.com",
    "settings": {
      "max_group_size": 100,
      "max_file_size_mb": 50,
      "enable_meetings": true,
      "enable_push_notifications": true
    }
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "comp_new001",
    "type": "companies",
    "attributes": {
      "unique_id": "comp_new001",
      "name": "Acme Corp",
      "url_id": "acme-corp",
      "domain": "acme.com",
      "logo_url": null,
      "settings": {
        "max_group_size": 100,
        "max_file_size_mb": 50,
        "enable_meetings": true,
        "enable_push_notifications": true
      },
      "plan": "free",
      "status": "active",
      "created_at": "2025-01-15T11:00:00Z",
      "updated_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

## API Keys

### List API Keys

Retrieve all API keys for a company.

**Endpoint:** `GET /companies/:url_id/keys`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| url_id | string | Yes | The company's URL identifier |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/companies/acme-corp/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "key_001",
      "type": "api_keys",
      "attributes": {
        "unique_id": "key_001",
        "name": "Production Key",
        "key_prefix": "pk_live_abc1",
        "environment": "production",
        "permissions": ["read", "write", "admin"],
        "last_used_at": "2025-01-15T10:00:00Z",
        "status": "active",
        "created_at": "2024-06-01T10:00:00Z",
        "expires_at": null
      }
    },
    {
      "id": "key_002",
      "type": "api_keys",
      "attributes": {
        "unique_id": "key_002",
        "name": "Development Key",
        "key_prefix": "pk_test_xyz9",
        "environment": "development",
        "permissions": ["read", "write"],
        "last_used_at": "2025-01-14T16:00:00Z",
        "status": "active",
        "created_at": "2024-06-01T10:30:00Z",
        "expires_at": null
      }
    }
  ]
}
```

---

### Create API Key

Generate a new API key for a company.

**Endpoint:** `POST /companies/:url_id/keys`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| url_id | string | Yes | The company's URL identifier |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Key name/label |
| environment | string | No | Environment: `production` (default), `development`, `staging` |
| permissions | array | No | Permissions: `read`, `write`, `admin` (default: `["read", "write"]`) |
| expires_at | string | No | ISO 8601 expiration date (null for no expiration) |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/companies/acme-corp/keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Staging Key",
    "environment": "staging",
    "permissions": ["read", "write"],
    "expires_at": "2025-12-31T23:59:59Z"
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "key_new001",
    "type": "api_keys",
    "attributes": {
      "unique_id": "key_new001",
      "name": "Staging Key",
      "key": "pk_stag_full_key_shown_only_once_abc123xyz789",
      "key_prefix": "pk_stag_full",
      "environment": "staging",
      "permissions": ["read", "write"],
      "status": "active",
      "created_at": "2025-01-15T12:00:00Z",
      "expires_at": "2025-12-31T23:59:59Z"
    }
  }
}
```

> **Note:** The full API key is only returned once at creation time. Store it securely.

---

### Update API Key

Update an existing API key's properties.

**Endpoint:** `PUT /companies/:url_id/keys/:key_unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| url_id | string | Yes | The company's URL identifier |
| key_unique_id | string | Yes | The API key unique ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | No | Updated key name |
| permissions | array | No | Updated permissions |
| status | string | No | Updated status: `active`, `revoked` |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/companies/acme-corp/keys/key_002" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Development Key - Updated",
    "permissions": ["read", "write", "admin"]
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "key_002",
    "type": "api_keys",
    "attributes": {
      "unique_id": "key_002",
      "name": "Development Key - Updated",
      "permissions": ["read", "write", "admin"],
      "updated_at": "2025-01-15T12:30:00Z"
    }
  }
}
```

---

### Delete API Key

Permanently revoke and delete an API key.

**Endpoint:** `DELETE /companies/:url_id/keys/:key_unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| url_id | string | Yes | The company's URL identifier |
| key_unique_id | string | Yes | The API key unique ID |

**cURL Example:**

```bash
curl -s -X DELETE "https://conversations.api.us.23blocks.com/companies/acme-corp/keys/key_002" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "key_002",
    "type": "api_keys",
    "attributes": {
      "unique_id": "key_002",
      "status": "revoked",
      "revoked_at": "2025-01-15T13:00:00Z"
    }
  }
}
```

---

## Key Exchange

### Exchange API Key

Exchange an API key from another 23blocks block to obtain credentials for this block. Enables cross-block communication.

**Endpoint:** `POST /companies/:url_id/exchange`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| url_id | string | Yes | The company's URL identifier |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| source_block | string | Yes | Source block name (e.g., `content-block`, `files-block`) |
| source_api_key | string | Yes | API key from the source block |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/companies/acme-corp/exchange" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "source_block": "content-block",
    "source_api_key": "pk_live_content_block_key_abc123"
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "exchange_001",
    "type": "key_exchanges",
    "attributes": {
      "conversations_api_key": "pk_live_conversations_exchanged_xyz789",
      "company_unique_id": "comp_001",
      "source_block": "content-block",
      "permissions": ["read", "write"],
      "expires_at": "2025-04-15T11:00:00Z",
      "created_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

## Impersonation

### Impersonate User

Generate an impersonation token to act on behalf of a user. Requires admin privileges.

**Endpoint:** `POST /companies/:url_id/impersonate`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| url_id | string | Yes | The company's URL identifier |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| user_unique_id | string | Yes | The user to impersonate |
| reason | string | Yes | Reason for impersonation (logged for audit) |
| duration | integer | No | Token duration in seconds (default: 3600, max: 86400) |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/companies/acme-corp/impersonate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "usr_abc123",
    "reason": "Debugging message delivery issue reported in ticket #4521",
    "duration": 1800
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "imp_001",
    "type": "impersonation_tokens",
    "attributes": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.imp...",
      "user_unique_id": "usr_abc123",
      "impersonated_by": "admin_user_001",
      "reason": "Debugging message delivery issue reported in ticket #4521",
      "expires_at": "2025-01-15T11:30:00Z",
      "created_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

> **Note:** Impersonation tokens should be used as the `Authorization: Bearer` header value. All actions performed with an impersonation token are logged in the audit trail.

---

## Data Model

### Company

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the company |
| name | string | Company name |
| url_id | string | URL-friendly identifier |
| domain | string | Company domain |
| logo_url | string | URL to company logo |
| settings | object | Company-specific settings |
| plan | string | Subscription plan: `free`, `starter`, `pro`, `enterprise` |
| status | string | Company status: `active`, `suspended`, `deleted` |
| created_at | datetime | Company registration timestamp |
| updated_at | datetime | Last update timestamp |

### API Key

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the key |
| name | string | Key name/label |
| key | string | Full API key (only shown at creation) |
| key_prefix | string | First characters of the key (for identification) |
| environment | string | Environment: `production`, `development`, `staging` |
| permissions | array | Permissions: `read`, `write`, `admin` |
| status | string | Key status: `active`, `revoked` |
| last_used_at | datetime | Last usage timestamp |
| expires_at | datetime | Expiration timestamp (null for no expiration) |
| created_at | datetime | Key creation timestamp |

### Key Exchange

| Field | Type | Description |
|-------|------|-------------|
| conversations_api_key | string | Exchanged API key for this block |
| company_unique_id | string | Associated company ID |
| source_block | string | Block that the source key came from |
| permissions | array | Permissions granted |
| expires_at | datetime | Exchange expiration |
| created_at | datetime | Exchange creation timestamp |

### Impersonation Token

| Field | Type | Description |
|-------|------|-------------|
| token | string | Impersonation bearer token |
| user_unique_id | string | User being impersonated |
| impersonated_by | string | Admin who initiated impersonation |
| reason | string | Audit reason for impersonation |
| expires_at | datetime | Token expiration |
| created_at | datetime | Token creation timestamp |

---

## Error Responses

### 401 Unauthorized

```json
{
  "errors": [
    {
      "status": "401",
      "title": "Unauthorized",
      "detail": "Invalid or missing authentication token"
    }
  ]
}
```

### 403 Forbidden

```json
{
  "errors": [
    {
      "status": "403",
      "title": "Forbidden",
      "detail": "Admin privileges required for impersonation"
    }
  ]
}
```

### 404 Not Found

```json
{
  "errors": [
    {
      "status": "404",
      "title": "Not Found",
      "detail": "Company with url_id 'invalid-corp' not found"
    }
  ]
}
```

### 409 Conflict

```json
{
  "errors": [
    {
      "status": "409",
      "title": "Conflict",
      "detail": "Company with url_id 'acme-corp' already exists"
    }
  ]
}
```

### 422 Unprocessable Entity

```json
{
  "errors": [
    {
      "status": "422",
      "title": "Unprocessable Entity",
      "detail": "Source API key is invalid or has been revoked"
    }
  ]
}
```
