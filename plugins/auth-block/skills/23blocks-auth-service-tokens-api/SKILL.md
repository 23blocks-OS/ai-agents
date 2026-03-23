---
name: 23blocks-auth-service-tokens-api
description: Manage 23blocks service tokens (M2M authentication) via REST API. Use when creating, listing, revoking, or regenerating machine-to-machine service tokens with categories (agent/service), scopes, and expiration policies.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Service Tokens API

Complete API reference for 23blocks machine-to-machine (M2M) service token management including creation, listing, revocation, and regeneration.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Auth API base URL | `https://auth.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://auth.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://auth.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### POST /companies/:company_url_id/service_tokens - Create Service Token

Creates a new M2M service token. Only human admin JWTs can create tokens (service tokens cannot self-create).

**Required Scope:** `service_tokens:create`

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/{company_url_id}/service_tokens" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-agent-token",
    "description": "Token for AI agent service",
    "token_category": "agent",
    "scopes": ["read", "write"],
    "expires_in_days": 30
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Token name |
| `description` | string | No | Token description |
| `token_category` | string | Yes | Category: `agent` or `service` |
| `scopes` | array | No | Requested scopes (must be subset of admin's scopes) |
| `expires_in_days` | integer | No | Expiration in days (max varies by category) |

**Response 201:**
```json
{
  "data": {
    "id": "token-uuid-123",
    "type": "service_token",
    "attributes": {
      "unique_id": "token-uuid-123",
      "name": "my-agent-token",
      "description": "Token for AI agent service",
      "token_category": "agent",
      "scopes": ["read", "write"],
      "status": "active",
      "expires_at": "2026-04-12T10:30:00Z",
      "created_at": "2026-03-13T10:30:00Z",
      "jwt": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Insufficient scope or scope escalation attempt
- `403 Forbidden` - Service token trying to self-create (blocked)
- `422 Unprocessable Entity` - Invalid category, token limit reached, max expiry exceeded, or validation failed

---

### GET /companies/:company_url_id/service_tokens - List Service Tokens

Lists service tokens for a company with optional filtering and pagination.

**Required Scope:** `service_tokens:read`

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/{company_url_id}/service_tokens?page=1&per_page=25&status=active&token_category=agent" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `per_page` | integer | No | Records per page (default: 25, max: 100) |
| `status` | string | No | Filter by status (e.g., `active`, `revoked`) |
| `token_category` | string | No | Filter by category (`agent` or `service`) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "token-uuid-123",
      "type": "service_token",
      "attributes": {
        "unique_id": "token-uuid-123",
        "name": "my-agent-token",
        "token_category": "agent",
        "status": "active",
        "scopes": ["read", "write"],
        "expires_at": "2026-04-12T10:30:00Z",
        "created_at": "2026-03-13T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /companies/:company_url_id/service_tokens/:unique_id - Show Service Token

Retrieves a single service token by ID.

**Required Scope:** `service_tokens:read`

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/{company_url_id}/service_tokens/{unique_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "token-uuid-123",
    "type": "service_token",
    "attributes": {
      "unique_id": "token-uuid-123",
      "name": "my-agent-token",
      "description": "Token for AI agent service",
      "token_category": "agent",
      "status": "active",
      "scopes": ["read", "write"],
      "expires_at": "2026-04-12T10:30:00Z",
      "created_at": "2026-03-13T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Service token not found

---

### DELETE /companies/:company_url_id/service_tokens/:unique_id - Revoke Service Token

Revokes a service token, permanently invalidating it.

**Required Scope:** `service_tokens:revoke`

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/companies/{company_url_id}/service_tokens/{unique_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "reason": "Token compromised"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reason` | string | No | Reason for revocation |

**Response 200:**
```json
{
  "data": {
    "id": "token-uuid-123",
    "type": "service_token",
    "attributes": {
      "status": "revoked"
    }
  }
}
```

---

### POST /companies/:company_url_id/service_tokens/:unique_id/regenerate - Regenerate Service Token

Atomically revokes the current token and creates a new one with the same configuration. Only human admin JWTs can regenerate tokens.

**Required Scope:** `service_tokens:create`

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/{company_url_id}/service_tokens/{unique_id}/regenerate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 201:**
```json
{
  "data": {
    "id": "new-token-uuid-456",
    "type": "service_token",
    "attributes": {
      "unique_id": "new-token-uuid-456",
      "name": "my-agent-token",
      "token_category": "agent",
      "status": "active",
      "jwt": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Token is not active

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "invalid_category",
    "title": "Invalid Category",
    "detail": "Token category must be one of: agent, service"
  }]
}
```

---

## Important Notes

- **Per-company cap**: There is a maximum number of active service tokens per company.
- **Scope escalation prevention**: Requested scopes must be a subset of the creating admin's scopes.
- **Self-creation blocked**: Service tokens cannot create or regenerate other service tokens; only human admin JWTs can.
- **Atomic regeneration**: Regenerate performs revoke + create in a single transaction.
