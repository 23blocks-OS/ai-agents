---
name: 23blocks-auth-sessions-api
description: Manage 23blocks sessions via REST API. Use when logging in, logging out, refreshing tokens, validating tokens, revoking tokens, or handling MFA during login.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "2.0"
  verified-by: 23blocks-api-authentication (Malachi)
  verified-date: "2026-05-14"
---

# Sessions API

Complete API reference for 23blocks session management including login, logout, token refresh, OAuth token management, and MFA.

> **Verified against Auth API codebase on 2026-05-14** by the Auth API team.

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

---

## Endpoints

### POST /auth/sign_in - Login

Authenticates a user and creates a new session.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/sign_in" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "secure_password"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email |
| `password` | string | Yes | User password |
| `mfa_code` | string | No | MFA code (if MFA enabled on account) |
| `backup_code` | string | No | Backup code (alternative to mfa_code) |

**Optional Headers:**
| Header | Description |
|--------|-------------|
| `X-OAuth-Mode: true` | Enables refresh token flow — response includes `refresh_token` |
| `X-Device-ID: <id>` | Associates session with a device for per-device token revocation |

**Response 200 (success):**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "user@example.com",
      "first_name": "John",
      "last_name": "Doe",
      "has_password": true
    }
  },
  "meta": {
    "auth": {
      "access_token": "eyJhbGciOiJSUzI1NiJ9...",
      "token_type": "Bearer",
      "expires_in": 86400,
      "expires_at": "2026-05-15T01:00:00Z",
      "scope": "scope1 scope2",
      "oauth_mode": false
    }
  }
}
```

**Response 200 (OAuth mode — with `X-OAuth-Mode: true`):**

Same as above, but `meta.auth` also includes:
```json
{
  "meta": {
    "auth": {
      "access_token": "eyJ...",
      "refresh_token": "refresh-token-string",
      "refresh_token_expires_in": 604800,
      "token_type": "Bearer",
      "expires_in": 86400,
      "oauth_mode": true
    }
  }
}
```

**Response 200 (MFA required):**
```json
{
  "errors": [{
    "code": "mfa_required",
    "meta": { "mfa_required": true }
  }]
}
```
> Re-send the login request with `mfa_code` or `backup_code` parameter.

**Errors:**
- `401 Unauthorized` - Invalid credentials or invalid MFA code (`{ errors: [{ code: "mfa_invalid" }] }`)
- `422 Unprocessable Entity` - Missing email or password
- `429 Too Many Requests` - Rate limit exceeded

---

### DELETE /auth/sign_out - Logout

Destroys the current session and invalidates the token.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/auth/sign_out" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Optional Headers:**
| Header | Description |
|--------|-------------|
| `X-Device-ID: <id>` | Revoke tokens for a specific device only |

**Optional Body:**
```json
{ "revoke_all": "true" }
```
> Revokes ALL refresh tokens for the user, not just the current session.

**Response 200:**
```json
{ "success": true, "message": "Signed out successfully" }
```

---

### POST /oauth/token/refresh - Refresh Token

Refreshes an expired access token using a refresh token. Requires OAuth mode to have been enabled during login.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/oauth/token/refresh" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "refresh_token": "refresh-token-string"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `refresh_token` | string | Yes | Valid refresh token from login |

**Response 200:** OAuth 2.0 standard token response with new access and refresh tokens.

**Errors:**
- `400 Bad Request` - `{ "error": "invalid_grant", "error_description": "..." }`

---

### POST /oauth/token/revoke - Revoke Token

Revokes a specific token. Always returns 200 per RFC 7009, even if the token doesn't exist.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/oauth/token/revoke" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "token": "token-to-revoke",
    "token_type_hint": "refresh_token"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | string | Yes | The token to revoke |
| `token_type_hint` | string | No | `refresh_token` or `access_token` |

**Response 200:**
```json
{ "revoked": true }
```

---

### POST /oauth/token/revoke_all - Revoke All Tokens

Revokes all tokens for a user, optionally scoped to a device.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/oauth/token/revoke_all" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "user-uuid-123",
    "device_id": "optional-device-id"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | string | Yes | User UUID |
| `device_id` | string | No | Scope revocation to a device |

**Response 200:**
```json
{ "revoked": true, "message": "All tokens revoked", "revoked_at": "2026-05-14T12:00:00Z" }
```

---

### GET /auth/validate_token - Validate Token

Verifies if the current access token is valid and returns the user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/auth/validate_token" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:** JSON:API user object (same shape as login `data`).

**Errors:**
- `401 Unauthorized` - Token is invalid or expired

---

### POST /auth/introspect - Token Introspection (RFC 7662)

Introspects a token to determine its state and metadata.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/introspect" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "token": "eyJhbGciOiJSUzI1NiJ9..."
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | string | Yes | Access token to introspect |

---

## JWKS & OIDC Discovery

For local JWT verification without calling the API:

```bash
# JWKS - tenant-scoped (USE THIS ONE)
curl "$BLOCKS_API_URL/<company_url_id>/.well-known/jwks.json"

# JWKS - app-scoped
curl "$BLOCKS_API_URL/apps/<app_url_id>/.well-known/jwks.json"

# OIDC Discovery
curl "$BLOCKS_API_URL/<company_url_id>/.well-known/openid-configuration"
```

> **Important:** The bare `/.well-known/jwks.json` (no tenant prefix) returns empty. Always use the tenant-scoped version.

---

## Error Response Format

All errors follow JSON:API format:
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

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-authentication
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

### Available Methods

```typescript
// AuthService — client.authentication.auth
signIn(request: SignInRequest): Promise<SignInResponse>;
signUp(request: SignUpRequest): Promise<SignUpResponse>;
signOut(): Promise<void>;
validateToken(): Promise<TokenValidationResponse>;
getCurrentUser(): Promise<User>;
requestPasswordReset(request: PasswordResetRequest): Promise<void>;
updatePassword(request: PasswordUpdateRequest): Promise<void>;
refreshToken(request: RefreshTokenRequest): Promise<RefreshTokenResponse>;
requestMagicLink(request: MagicLinkRequest): Promise<void>;
verifyMagicLink(request: MagicLinkVerifyRequest): Promise<SignInResponse>;
sendInvitation(request: InvitationRequest): Promise<void>;
acceptInvitation(request: AcceptInvitationRequest): Promise<SignInResponse>;
confirmEmail(token: string): Promise<User>;
resendConfirmation(request: ResendConfirmationRequest): Promise<void>;
validateEmail(request: ValidateEmailRequest): Promise<ValidateEmailResponse>;
validateDocument(request: ValidateDocumentRequest): Promise<ValidateDocumentResponse>;
resendInvitation(request: ResendInvitationRequest): Promise<User>;
requestAccountRecovery(request: AccountRecoveryRequest): Promise<AccountRecoveryResponse>;
completeAccountRecovery(request: CompleteRecoveryRequest): Promise<User>;
```

### React Hook

```typescript
import { useAuthenticationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAuthenticationBlock();

  // Example: Sign in a user
  const { user, accessToken } = await client.authentication.auth.signIn({
    email: 'user@example.com',
    password: 'password',
  });
}
```

---

## reCAPTCHA Enforcement

Tenants can enable inline reCAPTCHA v3 on auth endpoints (registration, login, passwordless). When enabled:
- Frontend must send `recaptcha_token` as a top-level body parameter
- Rate limit: 5 registrations per IP per 5 minutes (Rack::Attack)
- Kill switch: set `recaptcha_enforcement` to `false` in tenant config

**Agents using AID grant_type are NOT affected** — agent token exchange bypasses reCAPTCHA.
