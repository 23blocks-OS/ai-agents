---
name: 23blocks-conversations-companies-api
description: "Conversations Block tenant admin: companies, API keys, cross-block key exchange, impersonation. Use when configuring a tenant."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Companies API

Multi-tenant company management for the Conversations Block. Manage company profiles, API keys, key exchange for cross-block communication, and impersonation for administrative operations.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://realtime.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-conversations`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.
