---
name: 23blocks-auth-oidc-api
description: Manage 23blocks OpenID Connect (OIDC) endpoints via REST API. Use for OIDC discovery, authorization, token exchange, userinfo, and JWKS endpoints. Supports both company-level and app-level OIDC.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# OIDC API

Complete API reference for 23blocks OpenID Connect endpoints including discovery, authorization, token exchange, userinfo, and JWKS. Supports both company-level and app-level OIDC configurations.

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

### GET /:company_url_id/.well-known/openid-configuration - OIDC Discovery

Returns the OpenID Connect discovery document for a company. Public endpoint, cached for 24 hours.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/{company_url_id}/.well-known/openid-configuration"
```

**Response 200:**
```json
{
  "issuer": "https://auth.api.us.23blocks.com/my-company",
  "authorization_endpoint": "https://auth.api.us.23blocks.com/my-company/oauth/authorize",
  "token_endpoint": "https://auth.api.us.23blocks.com/my-company/oauth/token",
  "userinfo_endpoint": "https://auth.api.us.23blocks.com/my-company/oauth/userinfo",
  "jwks_uri": "https://auth.api.us.23blocks.com/my-company/.well-known/jwks.json",
  "introspection_endpoint": "https://auth.api.us.23blocks.com/my-company/oauth/introspect",
  "revocation_endpoint": "https://auth.api.us.23blocks.com/my-company/oauth/revoke",
  "response_types_supported": ["code", "token", "id_token", "code token", "code id_token", "token id_token", "code token id_token"],
  "grant_types_supported": ["authorization_code", "implicit", "password", "client_credentials", "refresh_token"],
  "subject_types_supported": ["public"],
  "id_token_signing_alg_values_supported": ["RS256"],
  "scopes_supported": ["openid", "profile", "email", "..."],
  "claims_supported": ["sub", "iss", "aud", "exp", "iat", "jti", "email", "name", "role", "scopes", "user_unique_id", "user_role_id"],
  "token_endpoint_auth_methods_supported": ["client_secret_post", "client_secret_basic"],
  "response_modes_supported": ["query", "fragment", "form_post"],
  "end_session_endpoint": "https://auth.api.us.23blocks.com/my-company/oauth/logout"
}
```

**Errors:**
- `404 Not Found` - Company not found or OIDC not enabled

---

### GET /apps/:app_url_id/.well-known/openid-configuration - App-Level OIDC Discovery

Returns the OIDC discovery document for an app. Same format as company-level but with `/apps/` prefix.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/apps/{app_url_id}/.well-known/openid-configuration"
```

**Response:** Same format as company-level discovery.

---

### GET /:company_url_id/oauth/userinfo - UserInfo

Returns OpenID Connect user information for the authenticated user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/{company_url_id}/oauth/userinfo" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN"
```

**Response 200:**
```json
{
  "sub": "user-uuid-123",
  "email": "user@example.com",
  "email_verified": true,
  "name": "John Doe",
  "preferred_username": "johndoe",
  "role": "admin",
  "user_role_id": 2,
  "updated_at": 1710324600,
  "given_name": "John",
  "family_name": "Doe",
  "locale": "en",
  "zoneinfo": "America/New_York"
}
```

**Notes:**
- `given_name`, `family_name`, `locale`, and `zoneinfo` are only included if the user has a profile.

**Errors:**
- `401 Unauthorized` - Missing or invalid Bearer token
- `404 Not Found` - User not found

---

### GET /apps/:app_url_id/oauth/userinfo - App-Level UserInfo

Same as company-level userinfo but accessed via app URL. Uses universal token validation.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/apps/{app_url_id}/oauth/userinfo" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN"
```

---

### GET /:company_url_id/oauth/authorize - Authorization Endpoint

Initiates the OAuth 2.0 authorization code flow. Redirects to login if not authenticated.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/{company_url_id}/oauth/authorize?client_id=my-client&response_type=code&redirect_uri=https://app.example.com/callback&scope=openid+profile+email&state=random-state&nonce=random-nonce"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `client_id` | string | Yes | OAuth client ID |
| `response_type` | string | Yes | Response type (e.g., `code`) |
| `redirect_uri` | string | Yes | Callback URL |
| `scope` | string | No | Requested scopes |
| `state` | string | No | CSRF state parameter |
| `nonce` | string | No | Replay protection nonce |

**Errors:**
- `400 Bad Request` (invalid_request) - Missing required parameters
- `403 Forbidden` - OIDC not enabled
- `404 Not Found` - Company not found

---

### POST /:company_url_id/oauth/token - Token Exchange

Exchanges authorization codes or refresh tokens for access tokens.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/{company_url_id}/oauth/token" \
  -H "Content-Type: application/json" \
  -d '{
    "grant_type": "authorization_code",
    "code": "auth-code-123",
    "redirect_uri": "https://app.example.com/callback",
    "client_id": "my-client",
    "client_secret": "my-secret"
  }'
```

**Supported Grant Types:**
| Grant Type | Description |
|------------|-------------|
| `authorization_code` | Exchange auth code for tokens |
| `refresh_token` | Exchange refresh token for new access token |
| `client_credentials` | Currently unavailable |

**Errors:**
- `400 Bad Request` (unsupported_grant_type) - Invalid grant type

---

### JWKS Endpoints

Public endpoints for retrieving JSON Web Key Sets.

**Company-level JWKS:**
```bash
curl -X GET "$BLOCKS_API_URL/{company_url_id}/.well-known/jwks.json"
```

**App-level JWKS:**
```bash
curl -X GET "$BLOCKS_API_URL/apps/{app_url_id}/.well-known/jwks.json"
```

**Company certificate:**
```bash
curl -X GET "$BLOCKS_API_URL/{company_url_id}/.well-known/certificate.pem"
```

**App certificate:**
```bash
curl -X GET "$BLOCKS_API_URL/apps/{app_url_id}/.well-known/certificate.pem"
```

**JWKS Health:**
```bash
curl -X GET "$BLOCKS_API_URL/.well-known/jwks-health"
```

**Global JWKS:**
```bash
curl -X GET "$BLOCKS_API_URL/.well-known/jwks.json"
```

**Response 200 (JWKS):**
```json
{
  "keys": [
    {
      "kty": "RSA",
      "kid": "key-uuid-123",
      "use": "sig",
      "alg": "RS256",
      "n": "...",
      "e": "AQAB"
    }
  ]
}
```

---

## Endpoint Summary

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/:company_url_id/.well-known/openid-configuration` | Public | OIDC discovery |
| GET | `/apps/:app_url_id/.well-known/openid-configuration` | Public | App OIDC discovery |
| GET | `/:company_url_id/oauth/userinfo` | Bearer | User info |
| GET | `/apps/:app_url_id/oauth/userinfo` | Bearer | App user info |
| GET | `/:company_url_id/oauth/authorize` | Public | Authorization flow |
| POST | `/:company_url_id/oauth/token` | Public | Token exchange |
| GET | `/:company_url_id/.well-known/jwks.json` | Public | Company JWKS |
| GET | `/apps/:app_url_id/.well-known/jwks.json` | Public | App JWKS |
| GET | `/.well-known/jwks.json` | Public | Global JWKS |
| GET | `/:company_url_id/.well-known/certificate.pem` | Public | Company certificate |
| GET | `/apps/:app_url_id/.well-known/certificate.pem` | Public | App certificate |

---

## Error Response Format

```json
{
  "error": "invalid_request",
  "error_description": "Missing required parameters: client_id, redirect_uri"
}
```

---

## Important Notes

- **OIDC must be enabled**: Discovery and authorization endpoints only work when OIDC is enabled for the company (toggle via Admin RSA Keys API).
- **RS256 signing**: All tokens are signed with RS256 algorithm using the company's RSA keys.
- **24-hour cache**: Discovery documents are cacheable for 24 hours.
- **App-level support**: All company-level endpoints have app-level equivalents under `/apps/:app_url_id/`.
- **Authorization code flow**: Currently returns `temporarily_unavailable` — use password or refresh_token grant types.
- **Client credentials**: Currently unavailable — service accounts should use the Service Tokens API instead.
