---
name: auth-sessions-api
description: Manage 23blocks sessions via REST API. Use when logging in, logging out, refreshing tokens, managing active sessions, verifying tokens, or handling MFA challenges.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Sessions API

Complete API reference for 23blocks session management including login, logout, token refresh, and MFA.

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
```bash
curl -X GET "$BLOCKS_API_URL/sessions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /sessions - Login

Authenticates a user and creates a new session.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/sessions" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "session": {
      "email": "user@example.com",
      "password": "secure_password"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email |
| `password` | string | Yes | User password |

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "session",
    "attributes": {
      "unique_id": "session-uuid-123",
      "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refresh_token": "refresh-token-string",
      "token_type": "Bearer",
      "expires_in": 3600,
      "created_at": "2025-01-12T10:30:00Z"
    },
    "relationships": {
      "user": {
        "data": { "id": "user-uuid-123", "type": "user" }
      }
    }
  }
}
```

**Errors:**
- `401 Unauthorized` - Invalid credentials
- `422 Unprocessable Entity` - Missing email or password
- `429 Too Many Requests` - Rate limit exceeded

---

### DELETE /sessions - Logout

Destroys the current session and invalidates the token.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/sessions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### POST /sessions/refresh - Refresh Token

Refreshes an expired access token using a refresh token.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/sessions/refresh" \
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
  "data": {
    "id": "session-uuid-456",
    "type": "session",
    "attributes": {
      "access_token": "new-access-token...",
      "refresh_token": "new-refresh-token",
      "token_type": "Bearer",
      "expires_in": 3600
    }
  }
}
```

**Errors:**
- `401 Unauthorized` - Invalid or expired refresh token

---

### GET /sessions - List Active Sessions

Lists all active sessions for the current user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/sessions?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "session-uuid-123",
      "type": "session",
      "attributes": {
        "unique_id": "session-uuid-123",
        "ip_address": "192.168.1.1",
        "user_agent": "Mozilla/5.0...",
        "last_active_at": "2025-01-12T10:30:00Z",
        "created_at": "2025-01-10T08:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 3
  }
}
```

---

### DELETE /sessions/:unique_id - Revoke Session

Revokes a specific session by ID.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/sessions/session-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Session not found

---

### POST /sessions/verify - Verify Token

Verifies if the current token is valid.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/sessions/verify" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "valid": true,
    "user_id": "user-uuid-123",
    "expires_at": "2025-01-12T11:30:00Z"
  }
}
```

**Errors:**
- `401 Unauthorized` - Token is invalid or expired

---

### POST /sessions/mfa/challenge - Initiate MFA Challenge

Initiates a multi-factor authentication challenge.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/sessions/mfa/challenge" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mfa": {
      "method": "totp"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `method` | string | Yes | MFA method (totp, sms, email) |

**Response 200:**
```json
{
  "data": {
    "challenge_id": "challenge-uuid-123",
    "method": "totp",
    "expires_at": "2025-01-12T10:35:00Z"
  }
}
```

---

### POST /sessions/mfa/verify - Verify MFA Code

Verifies the MFA code to complete authentication.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/sessions/mfa/verify" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mfa": {
      "challenge_id": "challenge-uuid-123",
      "code": "123456"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `challenge_id` | string | Yes | Challenge ID from initiate |
| `code` | string | Yes | MFA verification code |

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "session",
    "attributes": {
      "access_token": "fully-authenticated-token...",
      "mfa_verified": true
    }
  }
}
```

**Errors:**
- `401 Unauthorized` - Invalid MFA code
- `422 Unprocessable Entity` - Challenge expired

---

## Data Models

### Session
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Session identifier |
| `access_token` | string | JWT access token |
| `refresh_token` | string | Refresh token |
| `token_type` | string | Token type (Bearer) |
| `expires_in` | integer | Token TTL in seconds |
| `ip_address` | string | Client IP address |
| `user_agent` | string | Client user agent |
| `last_active_at` | timestamp | Last activity time |
| `created_at` | timestamp | Session creation time |

### MFAChallenge
| Field | Type | Description |
|-------|------|-------------|
| `challenge_id` | uuid | Challenge identifier |
| `method` | string | MFA method (totp, sms, email) |
| `expires_at` | timestamp | Challenge expiration |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "401",
    "code": "unauthorized",
    "title": "Authentication Failed",
    "detail": "Invalid email or password."
  }]
}
```
