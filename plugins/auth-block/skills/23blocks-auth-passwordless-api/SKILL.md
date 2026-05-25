---
name: 23blocks-auth-passwordless-api
description: Manage 23blocks passwordless login via REST API. Use when authenticating users with a 6-digit OTP code sent to their email, without requiring a password. Returns a full-scope JWT identical to password-based login.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "2.0"
  verified-by: 23blocks-api-authentication
  verified-date: "2026-05-16"
---

# Passwordless Login API

Complete API reference for 23blocks passwordless authentication. Allows users to sign in using a 6-digit OTP code delivered to their email, with no password required. Returns a full-scope JWT identical to standard password login.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Auth API base URL | `https://auth.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

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

> **Note:** Both passwordless endpoints (`/request` and `/verify`) only require the `X-API-KEY` header. Bearer auth is NOT needed for these endpoints.

---

## Endpoints

### POST /auth/passwordless/request - Send OTP Code

Sends a 6-digit OTP code to the user's email for passwordless login. The code expires in 10 minutes.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/passwordless/request" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
    "type": "passwordless",
    "attributes": {
      "status": "otp_sent",
      "email_hint": "u***@example.com",
      "expires_in": 600
    }
  }
}
```

**Security Notes:**
- Always returns 200 regardless of whether the email exists (anti-enumeration).
- OAuth-only users are silently skipped with no error leak.
- OTP codes are stored as SHA256 hashes, never in plaintext.

---

### POST /auth/passwordless/register - Request Registration OTP (v4.39)

Starts a passwordless-first registration flow. User registers with email + name only, never sets a password.

> This endpoint is for **human users only**. AI agents should use the AID protocol (`23blocks-auth-agent-identity-api`).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/passwordless/register" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "first_name": "Jane",
    "last_name": "Doe"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User's email address |
| `first_name` | string | Yes | First name |
| `last_name` | string | Yes | Last name |

**Response 200:**
```json
{
  "data": {
    "id": null,
    "type": "passwordless",
    "attributes": {
      "status": "otp_sent",
      "email_hint": "n***@example.com",
      "expires_in": 600
    }
  }
}
```

---

### POST /auth/passwordless/register/verify - Verify OTP and Create User (v4.39)

Verifies the OTP code and creates the user account. Returns a full-scope JWT. The created user has `has_password: false`.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/passwordless/register/verify" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "code": "123456"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Email used in registration request |
| `code` | string | Yes | 6-digit OTP code from email |

**Response 201 (Created):**
```json
{
  "data": {
    "id": "user-uuid-456",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-456",
      "email": "newuser@example.com",
      "first_name": "Jane",
      "last_name": "Doe",
      "has_password": false
    }
  },
  "meta": {
    "auth": {
      "access_token": "eyJhbGciOiJSUzI1NiJ9...",
      "token_type": "Bearer",
      "expires_in": 3600
    }
  }
}
```

> Future logins use the existing `POST /auth/passwordless/request` + `/verify` flow.

---

### POST /auth/passwordless/verify - Verify OTP and Get JWT

Validates the 6-digit OTP code and returns a full-scope JWT along with user data. Identical in shape to `POST /auth/sign_in`.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/passwordless/verify" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "code": "123456"
  }'
```

**Request with MFA (when MFA is enabled on the account):**
```bash
curl -X POST "$BLOCKS_API_URL/auth/passwordless/verify" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "code": "123456",
    "mfa_code": "789012"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User's email address |
| `code` | string | Yes | 6-digit OTP code received via email |
| `mfa_code` | string | No | TOTP code if MFA is enabled on account |
| `backup_code` | string | No | Backup code (alternative to `mfa_code`) |

**Response 200 (Success):**
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
      "expires_in": 3600
    }
  }
}
```

**Response 200 (MFA Challenge):**
```json
{
  "errors": [{
    "status": "200",
    "source": "Passwordless",
    "code": "mfa_required",
    "title": "MFA Required",
    "detail": "Multi-factor authentication is required. Re-send with mfa_code or backup_code."
  }]
}
```

**Errors:**
- `400 Bad Request` (GW-PL-011) - Invalid OTP code (includes `remaining_attempts` in meta)
- `400 Bad Request` (GW-PL-012) - Code expired
- `400 Bad Request` (GW-PL-013) - No pending code
- `401 Unauthorized` (GW-PL-014) - Max attempts exceeded (5 attempts)
- `401 Unauthorized` (mfa_invalid) - Wrong MFA code
- `500 Internal Server Error` (GW-PL-020) - Token generation failed

---

## Passwordless Flow

```
1. Client calls POST /auth/passwordless/request with email
2. Server sends 6-digit OTP to user's email (10 min expiry)
3. Client calls POST /auth/passwordless/verify with email + code
4. If MFA enabled: server returns mfa_required, client re-sends with mfa_code
5. On success: returns full-scope JWT (same as POST /auth/sign_in)
6. Client uses JWT for all authenticated API calls
```

---

## Error Response Format

```json
{
  "errors": [{
    "status": "400",
    "source": "Passwordless",
    "code": "GW-PL-011",
    "title": "Invalid Code",
    "detail": "The OTP code provided is invalid."
  }],
  "meta": {
    "remaining_attempts": 3
  }
}
```

---

## Use Cases

- **Mobile apps without password**: Allow mobile users to authenticate quickly via email OTP instead of remembering passwords.
- **Passwordless re-auth for sensitive actions**: Require a fresh OTP verification before performing high-risk operations (e.g., changing email, deleting account) without forcing users to recall their password.
- **Magic-link alternative for non-email channels**: Use the OTP code as a channel-agnostic alternative to magic links -- the 6-digit code can be delivered via SMS, push notification, or other transports beyond email.

---

## Important Notes

- **`has_password` field (v4.39)**: User responses now include `has_password: boolean`. Passwordless-registered users have `false`. Detection: `user.provider === 'email' && user.has_password === false` means truly passwordless.
- **Forgot password redirect (v4.39)**: When `POST /auth/password` is called for a passwordless user, it returns the same OTP envelope as `/auth/passwordless/request` instead of the classic Devise reset-link. Frontends can use one handler.
- **Full-scope JWT**: Unlike password OTP (which returns a scoped `password:reset` token), passwordless verify returns a full-scope JWT identical to `POST /auth/sign_in` -- same scopes, same lifetime, same RS256 signing.
- **Max 5 attempts**: After 5 failed verification attempts, the OTP is cleared and a new one must be requested.
- **10-minute expiry**: OTP codes expire after 10 minutes.
- **Timing-safe comparison**: Code verification uses constant-time comparison to prevent timing attacks.
- **Anti-enumeration**: The request endpoint always returns 200, never revealing whether an email exists.
- **OAuth users silently skipped**: No error is returned for OAuth-only accounts, preventing information leakage.
- **MFA gate**: MFA handling mirrors `POST /auth/sign_in` exactly -- same challenge/response pattern.
- **No migration needed**: No Mandrill template required; inline text fallback works automatically.
- **reCAPTCHA**: Tenants with `recaptcha_enforcement` enabled require a `recaptcha_token` top-level body param. Agents using AID grant_type are NOT affected.
