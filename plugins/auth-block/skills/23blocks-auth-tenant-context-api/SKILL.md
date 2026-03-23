---
name: 23blocks-auth-tenant-context-api
description: Manage 23blocks tenant context switching via REST API. Use when switching between multi-tenant companies, revoking tenant contexts, or auditing context switch history.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Tenant Context API

Complete API reference for 23blocks secure multi-tenant context switching with audit trail. Allows users to switch between companies/tenants they have access to, generating short-lived tenant context tokens.

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

### POST /auth/tenant-context - Switch Tenant Context

Creates a short-lived tenant context token for the specified company. The user must have access to the target company.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/tenant-context" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tenant_context": {
      "company_url_id": "my-company",
      "switch_reason": "app_selection"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tenant_context.company_url_id` | string | Yes* | Company URL ID to switch to |
| `tenant_context.company_id` | string | Yes* | Company unique ID (alternative to url_id) |
| `tenant_context.app_url_id` | string | Yes* | App URL ID (alternative to url_id) |
| `tenant_context.switch_reason` | string | No | Reason for switch (default: `app_selection`) |

*One of `company_url_id`, `company_id`, or `app_url_id` is required.

**Response 200:**
```json
{
  "data": {
    "type": "tenant_context",
    "id": "audit-record-id",
    "attributes": {
      "tenant_context_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires_in": 1800,
      "expires_at": "2026-03-13T11:00:00Z",
      "tenant_info": {
        "company_id": "company-uuid-123",
        "company_name": "My Company",
        "company_url_id": "my-company",
        "role_id": 2,
        "role_name": "admin",
        "permissions": ["read", "write", "admin:write"],
        "schema_name": "tenant_my_company"
      },
      "audit_id": "audit-record-id"
    }
  }
}
```

**Errors:**
- `401 Unauthorized` - Authentication required
- `403 Forbidden` - User does not have access to the target company
- `404 Not Found` - Company not found
- `429 Too Many Requests` - Rate limit exceeded (max 150 switches/hour)
- `500 Internal Server Error` - Context creation failed

---

### POST /auth/tenant-context/revoke - Revoke Tenant Context

Revokes an active tenant context token.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/tenant-context/revoke" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tenant_context": {
      "tenant_context_token_id": "token-jti-uuid"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tenant_context.tenant_context_token_id` | string | Yes | The JTI of the tenant context token to revoke |

**Response 200:**
```json
{
  "message": "Tenant context revoked successfully"
}
```

**Errors:**
- `404 Not Found` - Context not found or already revoked

---

### GET /auth/tenant-context/audit - Audit Context Switches

Lists the last 50 tenant context switches for the current user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/auth/tenant-context/audit" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "type": "tenant_context_audit",
      "id": "audit-id-123",
      "attributes": {
        "company_name": "My Company",
        "company_url_id": "my-company",
        "switch_reason": "app_selection",
        "created_at": "2026-03-13T10:30:00Z",
        "expires_at": "2026-03-13T11:00:00Z",
        "revoked": false,
        "revoked_at": null,
        "ip_address": "192.168.1.1",
        "active": true
      }
    }
  ]
}
```

---

## Error Response Format

```json
{
  "errors": [{
    "status": "403",
    "code": "10501",
    "title": "Access Denied",
    "detail": "User does not have access to this tenant."
  }]
}
```

---

## Important Notes

- **30-minute TTL**: Tenant context tokens expire after 30 minutes.
- **Rate limiting**: Maximum 150 context switches per hour per user.
- **Audit trail**: All context switches are recorded with IP address, user agent, and reason.
- **Role-based**: The tenant context token includes the user's role and permissions for the target company.
- **Revocable**: Active tenant contexts can be explicitly revoked before expiration.
