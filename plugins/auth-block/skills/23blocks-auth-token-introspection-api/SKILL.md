---
name: 23blocks-auth-token-introspection-api
description: Introspect 23blocks tokens via REST API. Use when validating tokens, checking token status, or retrieving token metadata following the OAuth 2.0 RFC 7662 standard.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Token Introspection API

Complete API reference for 23blocks OAuth 2.0 token introspection (RFC 7662 compliant). Always returns HTTP 200 with `active: true/false`.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://auth.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Auth API base URL | `https://auth.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token to introspect | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

The token to introspect can be passed via Bearer header, `Company-Token` header, or `token` body parameter. The `AppId` header is required.

```bash
curl -X POST "$BLOCKS_API_URL/auth/introspect" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /auth/introspect - Introspect Token

Validates a token and returns its metadata. Per RFC 7662, always returns HTTP 200 with `active: true` or `active: false`.

**Request (via Bearer header):**
```bash
curl -X POST "$BLOCKS_API_URL/auth/introspect" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Request (via body parameter):**
```bash
curl -X POST "$BLOCKS_API_URL/auth/introspect" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
  }'
```

**Token Sources (checked in order):**
1. `Authorization: Bearer <token>` header
2. `Company-Token` header
3. `token` body parameter

**Response 200 (active token):**
```json
{
  "active": true,
  "user_unique_id": "user-uuid-123",
  "user_role_id": 2,
  "company_id": "company-uuid-456",
  "scopes": ["read", "write", "admin:write"],
  "expires_at": "2026-03-13T11:30:00Z",
  "issued_at": "2026-03-13T10:30:00Z",
  "issuer": "23blocks-auth"
}
```

**Response 200 (inactive token):**
```json
{
  "active": false,
  "errors": [
    {
      "status": 200,
      "source": "Token Introspection",
      "code": "token_expired",
      "title": "Token Expired",
      "detail": "The provided token has expired"
    }
  ]
}
```

**Error Codes (all return HTTP 200 with `active: false`):**
| Code | Title | Description |
|------|-------|-------------|
| `missing_token` | Missing Token | No token provided in request |
| `missing_api_key` | Missing API Key | No API key in request |
| `invalid_api_key` | Invalid API Key | API key not found |
| `token_expired` | Token Expired | Token has expired |
| `invalid_token` | Invalid Token | Token is invalid or malformed |
| `validation_error` | Validation Error | Error during validation |
| `validation_failed` | Token Validation Failed | Unexpected error |

---

## Error Response Format

Per RFC 7662, error responses always return HTTP 200 with `active: false`:

```json
{
  "active": false,
  "errors": [{
    "status": 200,
    "source": "Token Introspection",
    "code": "invalid_token",
    "title": "Invalid Token",
    "detail": "The provided token is invalid or malformed"
  }]
}
```

---

## Important Notes

- **RFC 7662 compliant**: Always returns HTTP 200, never 401/403. Use the `active` field to determine token validity.
- **No authentication required**: The endpoint itself does not require authentication; it validates the provided token.
- **API key required**: The `AppId` header must be provided to identify the company for token validation.
- **Universal validation**: Uses `validate_token_universal` which supports multiple key formats and rotation.
