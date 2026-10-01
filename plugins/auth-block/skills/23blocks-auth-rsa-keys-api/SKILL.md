---
name: 23blocks-auth-rsa-keys-api
description: "Auth Block admin RSA keys: rotate, certificates, OIDC toggle, JWKS preview. Use when rotating signing keys. Admin only."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# RSA Keys API (Admin)

Complete API reference for 23blocks admin RSA key management including key rotation, certificate generation, OIDC toggle, and JWKS preview.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://auth.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /admin/rsa_keys - Key Status

Returns current RSA key information and OIDC configuration for the company.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/admin/rsa_keys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "rsa_keys_info",
    "id": 1,
    "attributes": {
      "jwt_key_id": "key-uuid-123",
      "certificate_expires_at": "2027-03-13T00:00:00Z",
      "key_rotation_started_at": null,
      "key_rotation_completed_at": "2026-01-15T10:00:00Z",
      "oidc_enabled": true,
      "oidc_issuer_url": "https://auth.api.us.23blocks.com/my-company",
      "has_valid_keys": true,
      "days_until_certificate_expires": 365,
      "rotation_in_progress": false
    }
  }
}
```

---

### POST /admin/rsa_keys/rotate - Start Key Rotation

Initiates RSA key rotation. Generates new keys while keeping old keys active during the transition period.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/admin/rsa_keys/rotate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "rsa_key_rotation",
    "id": 1,
    "attributes": {
      "status": "started",
      "new_key_id": "new-key-uuid-456",
      "previous_key_id": "key-uuid-123",
      "started_at": "2026-03-13T10:30:00Z",
      "message": "Key rotation initiated successfully"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Key rotation already in progress

---

### POST /admin/rsa_keys/complete_rotation - Complete Key Rotation

Completes an in-progress key rotation, removing the previous key reference.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/admin/rsa_keys/complete_rotation" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "rsa_key_rotation",
    "id": 1,
    "attributes": {
      "status": "completed",
      "key_id": "new-key-uuid-456",
      "completed_at": "2026-03-13T10:35:00Z",
      "message": "Key rotation completed successfully"
    }
  }
}
```

**Errors:**
- `400 Bad Request` - No key rotation in progress

---

### POST /admin/rsa_keys/generate_certificate - Generate Certificate

Generates a new X.509 certificate for the current RSA keys.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/admin/rsa_keys/generate_certificate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "certificate_generation",
    "id": 1,
    "attributes": {
      "certificate_expires_at": "2027-03-13T00:00:00Z",
      "message": "Certificate generated successfully"
    }
  }
}
```

**Errors:**
- `500 Internal Server Error` - Certificate generation failed

---

### POST /admin/rsa_keys/toggle_oidc - Toggle OIDC

Enables or disables OpenID Connect for the company.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/admin/rsa_keys/toggle_oidc" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `enabled` | boolean | Yes | `true` to enable, `false` to disable OIDC |

**Response 200:**
```json
{
  "data": {
    "type": "oidc_configuration",
    "id": 1,
    "attributes": {
      "oidc_enabled": true,
      "oidc_issuer_url": "https://auth.api.us.23blocks.com/my-company",
      "message": "OIDC enabled successfully"
    }
  }
}
```

---

### GET /admin/rsa_keys/jwks_preview - Preview JWKS

Previews the current JWKS (JSON Web Key Set) document and related URLs.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/admin/rsa_keys/jwks_preview" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "jwks_preview",
    "id": 1,
    "attributes": {
      "jwks": {
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
      },
      "jwks_url": "https://auth.api.us.23blocks.com/my-company/.well-known/jwks.json",
      "oidc_discovery_url": "https://auth.api.us.23blocks.com/my-company/.well-known/openid-configuration"
    }
  }
}
```

**Errors:**
- `404 Not Found` - No valid RSA keys available

---

## Key Rotation Flow

```
1. GET  /admin/rsa_keys                  → Check current key status
2. POST /admin/rsa_keys/rotate           → Start rotation (new keys generated)
3. Wait for clients to pick up new JWKS  → Allow propagation time
4. POST /admin/rsa_keys/complete_rotation → Finalize rotation (remove old key ref)
```

---

## Error Response Format

```json
{
  "error": "Key rotation already in progress",
  "started_at": "2026-03-13T10:30:00Z"
}
```

---

## Important Notes

- **Admin only**: All endpoints require `admin:write` scope or admin role.
- **Two-phase rotation**: Key rotation is a two-step process (start + complete) to allow client propagation time.
- **No concurrent rotation**: Only one rotation can be in progress at a time.
- **OIDC dependency**: OIDC discovery endpoints (`.well-known/openid-configuration`) only work when OIDC is enabled.
- **Certificate management**: Certificates are generated independently from key rotation.
