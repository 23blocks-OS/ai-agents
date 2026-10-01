---
name: 23blocks-auth-oauth-api
description: "Auth Block OAuth/SSO: configure Google, GitHub and other providers, authorization URLs, callbacks, SSO. Use for social login or SSO setup."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# OAuth / SSO API

Complete API reference for 23blocks OAuth provider configuration and SSO login.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://auth.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /oauth/providers - List OAuth Providers

Lists all configured OAuth providers.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/oauth/providers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "provider-uuid-123",
      "type": "oauth_provider",
      "attributes": {
        "unique_id": "provider-uuid-123",
        "name": "google",
        "display_name": "Google",
        "enabled": true,
        "scopes": ["email", "profile"],
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /oauth/providers - Configure Provider

Configures a new OAuth provider.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/oauth/providers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "provider": {
      "name": "github",
      "client_id": "your-github-client-id",
      "client_secret": "your-github-client-secret",
      "redirect_uri": "https://yourapp.com/auth/github/callback",
      "scopes": ["user:email", "read:user"]
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Provider name (google, github, etc.) |
| `client_id` | string | Yes | OAuth client ID |
| `client_secret` | string | Yes | OAuth client secret |
| `redirect_uri` | string | Yes | Callback URL |
| `scopes` | array | No | Requested OAuth scopes |

**Response 201:**
```json
{
  "data": {
    "id": "provider-uuid-456",
    "type": "oauth_provider",
    "attributes": {
      "unique_id": "provider-uuid-456",
      "name": "github",
      "display_name": "GitHub",
      "enabled": true,
      "scopes": ["user:email", "read:user"],
      "redirect_uri": "https://yourapp.com/auth/github/callback",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Provider already configured
- `422 Unprocessable Entity` - Validation errors

---

### PUT /oauth/providers/:provider_id - Update Provider

Updates an existing OAuth provider configuration.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/oauth/providers/provider-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "provider": {
      "redirect_uri": "https://newapp.com/auth/google/callback",
      "scopes": ["email", "profile", "openid"],
      "enabled": true
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "provider-uuid-123",
    "type": "oauth_provider",
    "attributes": {
      "unique_id": "provider-uuid-123",
      "name": "google",
      "redirect_uri": "https://newapp.com/auth/google/callback",
      "scopes": ["email", "profile", "openid"],
      "enabled": true,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /oauth/providers/:provider_id - Remove Provider

Removes an OAuth provider configuration.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/oauth/providers/provider-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Provider not found

---

### GET /oauth/authorize/:provider - Get Authorization URL

Generates an OAuth authorization URL for the specified provider.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/oauth/authorize/google?redirect_uri=https://yourapp.com/auth/callback" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `redirect_uri` | string | No | Override default redirect URI |
| `state` | string | No | Custom state parameter |

**Response 200:**
```json
{
  "data": {
    "authorization_url": "https://accounts.google.com/o/oauth2/v2/auth?client_id=...&redirect_uri=...&state=...",
    "state": "random-state-string",
    "provider": "google"
  }
}
```

---

### POST /oauth/callback/:provider - Handle OAuth Callback

Handles the OAuth provider callback after user authorization.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/oauth/callback/google" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "authorization-code-from-google",
    "state": "state-parameter"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Authorization code from provider |
| `state` | string | Yes | State parameter for CSRF validation |

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "session",
    "attributes": {
      "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refresh_token": "refresh-token-string",
      "token_type": "Bearer",
      "expires_in": 3600,
      "is_new_user": false
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
- `401 Unauthorized` - Invalid authorization code
- `422 Unprocessable Entity` - State mismatch or provider error

---

### POST /sso/login - SSO Login

Initiates a SAML/SSO login flow.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/sso/login" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "sso": {
      "email": "user@enterprise.com"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email to determine SSO provider |

**Response 200:**
```json
{
  "data": {
    "sso_url": "https://idp.enterprise.com/saml/sso?SAMLRequest=...",
    "provider": "enterprise_saml",
    "binding": "redirect"
  }
}
```

**Errors:**
- `404 Not Found` - No SSO configured for email domain
- `422 Unprocessable Entity` - Invalid email format

---

### GET /sso/config - Get SSO Configuration

Retrieves the current SSO configuration.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/sso/config" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "sso-config-uuid",
    "type": "sso_config",
    "attributes": {
      "enabled": true,
      "provider_type": "saml",
      "idp_entity_id": "https://idp.enterprise.com",
      "idp_sso_url": "https://idp.enterprise.com/saml/sso",
      "idp_certificate": "-----BEGIN CERTIFICATE-----...",
      "sp_entity_id": "https://auth.api.us.23blocks.com/sso",
      "allowed_domains": ["enterprise.com"],
      "auto_provision": true,
      "default_role": "member"
    }
  }
}
```

---

## Data Models

### OAuthProvider
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Provider name (google, github, etc.) |
| `display_name` | string | Human-readable name |
| `client_id` | string | OAuth client ID |
| `redirect_uri` | string | Callback URL |
| `scopes` | array | Requested OAuth scopes |
| `enabled` | boolean | Whether provider is active |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### SSOConfig
| Field | Type | Description |
|-------|------|-------------|
| `enabled` | boolean | Whether SSO is active |
| `provider_type` | string | SSO type (saml, oidc) |
| `idp_entity_id` | string | Identity provider entity ID |
| `idp_sso_url` | string | SSO login URL |
| `idp_certificate` | string | IdP X.509 certificate |
| `sp_entity_id` | string | Service provider entity ID |
| `allowed_domains` | array | Whitelisted email domains |
| `auto_provision` | boolean | Auto-create users on first login |
| `default_role` | string | Default role for provisioned users |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"401","code":"oauth_error","title":"OAuth Authentication Failed","detail":"The authorization code is invalid or has expired."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-authentication`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// OAuthService — client.authentication.oauth
facebookLogin(request: OAuthSocialLoginRequest): Promise<SignInResponse>;
googleLogin(request: OAuthSocialLoginRequest): Promise<SignInResponse>;
tenantLogin(request: TenantLoginRequest): Promise<SignInResponse>;
introspectToken(token?: string): Promise<TokenIntrospectionResponse>;
refreshToken(refreshToken: string): Promise<{ accessToken: string; refreshToken?: string; tokenType: string; expiresIn?: number }>;
revokeToken(request: TokenRevokeRequest): Promise<TokenRevokeResponse>;
revokeAllTokens(request: TokenRevokeAllRequest): Promise<TokenRevokeResponse>;
createTenantContext(request: TenantContextCreateRequest): Promise<TenantContextResponse>;
revokeTenantContext(request: TenantContextRevokeRequest): Promise<{ message: string }>;
getTenantContextAudit(): Promise<TenantContextAuditEntry[]>;
```

### TypeScript Types

```typescript
import type {
  OAuthSocialLoginRequest,
  TenantLoginRequest,
  TokenIntrospectionResponse,
  TokenRevokeRequest,
  TokenRevokeAllRequest,
  TokenRevokeResponse,
  TenantContextCreateRequest,
  TenantInfo,
  TenantContextResponse,
  TenantContextRevokeRequest,
  TenantContextAuditEntry,
  SignInResponse,
} from '@23blocks/block-authentication';
```

### React Hook

```typescript
import { useAuthenticationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAuthenticationBlock();

  // Example: Login with Google
  const { user, accessToken } = await client.authentication.oauth.googleLogin({
    token: 'google-access-token',
  });
}
```
