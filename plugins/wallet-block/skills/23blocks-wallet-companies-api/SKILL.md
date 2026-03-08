---
name: 23blocks-wallet-companies-api
description: Manage 23blocks Wallet Block companies via REST API. Use when creating companies, managing API keys, configuring exchange settings, impersonating users, or creating storage configurations.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Companies API

Complete API reference for 23blocks Wallet Block multi-tenant company management with API keys, exchange settings, user impersonation, and storage configuration.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

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

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/companies/:url_id` | Get company details |
| POST | `/companies/` | Create a new company |
| GET | `/companies/:url_id/keys` | List API keys |
| POST | `/companies/:unique_id/keys` | Add API key |
| DELETE | `/companies/:unique_id/keys/:key_unique_id` | Delete API key |
| POST | `/companies/:unique_id/exchange` | Add exchange rate settings |
| POST | `/companies/:url_id/access` | Impersonate a user |
| POST | `/companies/:url_id/storage` | Create storage configuration |

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
