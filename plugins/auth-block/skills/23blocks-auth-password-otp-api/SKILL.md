---
name: 23blocks-auth-password-otp-api
description: Manage 23blocks OTP-based password reset via REST API. Use when requesting a 6-digit OTP code for password reset or verifying OTP codes to obtain scoped JWT tokens. App-friendly flow with no redirects.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Password OTP API

Complete API reference for 23blocks OTP-based password reset. Provides an app-friendly, redirect-free flow using 6-digit OTP codes with 10-minute expiry and 5 maximum attempts.

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

These endpoints require the `AppId` header but do NOT require a Bearer token (public endpoints for password reset flow).

```bash
curl -X POST "$BLOCKS_API_URL/auth/password/otp" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /auth/password/otp - Request OTP Code

Sends a 6-digit OTP code to the user's email for password reset. The code expires in 10 minutes.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/password/otp" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User's email address |

**Response 200:**
```json
{
  "data": {
    "id": null,
    "type": "password_otp",
    "attributes": {
      "status": "otp_sent",
      "email_hint": "u***@example.com",
      "expires_in": 600
    }
  },
  "meta": {
    "message": "If an account exists with this email, a password reset code has been sent."
  }
}
```

**Errors:**
- `400 Bad Request` (GW-POTP-001) - Missing email
- `404 Not Found` (GW-POTP-004) - User not found (non-paranoid mode only)
- `422 Unprocessable Entity` (GW-POTP-005) - OAuth user (password not applicable)

**Security Notes:**
- In paranoid mode (default), the response is identical whether the email exists or not, preventing email enumeration.
- Only users with `provider: email` and `status: active` are eligible for OTP reset.
- OTP codes are stored as SHA256 hashes, never in plaintext.

---

### POST /auth/password/otp/verify - Verify OTP Code

Verifies the 6-digit OTP code and returns a scoped JWT token for password reset along with user data.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/password/otp/verify" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "code": "123456"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User's email address |
| `code` | string | Yes | 6-digit OTP code received via email |

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "user@example.com",
      "name": "John Doe"
    },
    "relationships": {
      "role": { "data": { "id": "1", "type": "role" } },
      "user_profile": { "data": { "id": "profile-uuid", "type": "user_profile" } }
    }
  },
  "meta": {
    "auth": {
      "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
      "token_type": "Bearer",
      "expires_in": 1800,
      "scope": "password:reset"
    }
  }
}
```

**Errors:**
- `400 Bad Request` (GW-POTP-010) - Missing email or code
- `400 Bad Request` (GW-POTP-011) - Invalid OTP code (includes `remaining_attempts` in meta)
- `400 Bad Request` (GW-POTP-012) - Code expired
- `400 Bad Request` (GW-POTP-013) - No pending OTP code
- `401 Unauthorized` (GW-POTP-014) - Max attempts exceeded (5 attempts)
- `404 Not Found` (GW-POTP-015) - User not found (non-paranoid mode)
- `500 Internal Server Error` (GW-POTP-020) - Token generation failed

---

## OTP Flow

```
1. Client calls POST /auth/password/otp with email
2. Server sends 6-digit OTP to user's email (10 min expiry)
3. Client calls POST /auth/password/otp/verify with email + code
4. On success: returns scoped JWT (scope: password:reset, 30 min TTL)
5. Client uses scoped JWT to call PUT /auth/password to set new password
```

---

## Error Response Format

```json
{
  "errors": [{
    "status": "400",
    "source": "Password OTP",
    "code": "GW-POTP-011",
    "title": "Invalid Code",
    "detail": "The OTP code provided is invalid."
  }],
  "meta": {
    "remaining_attempts": 3
  }
}
```

---

## Important Notes

- **Max 5 attempts**: After 5 failed verification attempts, the OTP is cleared and a new one must be requested.
- **10-minute expiry**: OTP codes expire after 10 minutes.
- **Timing-safe comparison**: Code verification uses constant-time comparison to prevent timing attacks.
- **Atomic increment**: Failed attempt counter uses atomic database increment to prevent race conditions.
- **Scoped JWT**: The returned token has `password:reset` scope and 30-minute TTL, only valid for password change operations.
