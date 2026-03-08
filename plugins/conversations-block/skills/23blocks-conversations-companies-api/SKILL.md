---
name: 23blocks-conversations-companies-api
description: Manage multi-tenant companies, API keys, cross-block key exchange, and user impersonation. Use when registering companies, managing API credentials, or enabling cross-block communication.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Companies API

Multi-tenant company management for the Conversations Block. Manage company profiles, API keys, key exchange for cross-block communication, and impersonation for administrative operations.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Conversations API base URL | `https://conversations.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

```bash
curl -X GET "$BLOCKS_API_URL/companies/{url_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/companies/:url_id` | Get company |
| POST | `/companies/` | Create company |
| GET | `/companies/:url_id/keys` | List API keys |
| POST | `/companies/:url_id/keys` | Create API key |
| PUT | `/companies/:url_id/keys/:key_unique_id` | Update API key |
| DELETE | `/companies/:url_id/keys/:key_unique_id` | Delete API key |
| POST | `/companies/:url_id/exchange` | Exchange API key |
| POST | `/companies/:url_id/impersonate` | Impersonate user |

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

## Error Response Format

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

Common status codes: `401` Unauthorized, `403` Forbidden, `404` Not Found, `409` Conflict, `422` Unprocessable Entity.

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-conversations
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
