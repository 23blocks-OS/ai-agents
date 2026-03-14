---
name: 23blocks-auth-token-management-api
description: Manage 23blocks OAuth 2.0 tokens via REST API. Use when refreshing access tokens, revoking individual tokens, or revoking all tokens for a user or device.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Token Management API

Complete API reference for 23blocks OAuth 2.0 token management including refresh, revoke, and revoke-all operations.

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
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

These endpoints use the `AppId` header for client identification. Bearer token is not required for token refresh/revoke operations.

```bash
curl -X POST "$BLOCKS_API_URL/oauth/token/refresh" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /oauth/token/refresh - Refresh Access Token

Exchanges a valid refresh token for a new access token.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/oauth/token/refresh" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "refresh_token": "refresh-token-string"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `refresh_token` | string | Yes | Valid refresh token |

**Response 200:**
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "new-refresh-token-string",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

**Errors:**
- `400 Bad Request` - Missing refresh_token parameter, missing or invalid API key
- `400 Bad Request` (invalid_grant) - Invalid or expired refresh token
- `400 Bad Request` (server_error) - Token refresh failed

---

### POST /oauth/token/revoke - Revoke Token

Revokes a specific token (refresh or access). Per OAuth 2.0 spec (RFC 7009), returns 200 even if the token does not exist.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/oauth/token/revoke" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "token": "token-to-revoke",
    "token_type_hint": "refresh_token"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | string | Yes | Token to revoke |
| `token_type_hint` | string | No | Hint: `refresh_token` or `access_token` |

**Response 200:**
```json
{
  "revoked": true
}
```

**Errors:**
- `400 Bad Request` - Missing token parameter, missing or invalid API key

**Note:** Access tokens are stateless JWTs and cannot be truly revoked. The endpoint returns success per RFC 7009 spec, but the access token remains valid until expiration.

---

### POST /oauth/token/revoke_all - Revoke All Tokens

Revokes all refresh tokens for a user, optionally filtered by device.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/oauth/token/revoke_all" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "user-uuid-123",
    "device_id": "device-uuid-456"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | string | Yes | User unique ID |
| `device_id` | string | No | Device ID to scope revocation (optional) |

**Response 200:**
```json
{
  "revoked": true,
  "message": "All tokens revoked for user user-uuid-123",
  "revoked_at": "2026-03-13T10:30:00Z"
}
```

**Errors:**
- `400 Bad Request` - Missing API key, missing user_unique_id, user not found

---

## Error Response Format

```json
{
  "error": "invalid_grant",
  "error_description": "The refresh token is invalid or expired."
}
```

---

## Important Notes

- **OAuth 2.0 compliant**: Follows RFC 7009 for token revocation.
- **Stateless access tokens**: Access tokens are JWTs that cannot be server-side revoked; only refresh tokens can be revoked.
- **Device-scoped revocation**: Use `device_id` to revoke tokens for a specific device without affecting other sessions.
- **Multi-tenant aware**: Token operations are scoped to the company identified by the API key.
