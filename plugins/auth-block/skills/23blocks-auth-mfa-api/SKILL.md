---
name: 23blocks-auth-mfa-api
description: Manage 23blocks multi-factor authentication (MFA) via REST API. Use when setting up TOTP authenticator apps, enabling/disabling MFA, verifying TOTP or backup codes, and checking MFA status.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# MFA API

Complete API reference for 23blocks multi-factor authentication management including TOTP setup, enable, disable, verify, and status checking with backup code support.

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
curl -X POST "$BLOCKS_API_URL/users/{unique_id}/mfa/setup" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Important:** All MFA endpoints require the authenticated user to be the same user identified by `:unique_id` in the URL. Users can only manage their own MFA.

---

## Endpoints

### POST /users/:unique_id/mfa/setup - Setup MFA

Generates a TOTP secret, QR code URI for authenticator apps, and 10 backup codes.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/{unique_id}/mfa/setup" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "regenerate": "false"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `regenerate` | string | No | Set to `"true"` to regenerate secret and reset MFA |

**Response 200:**
```json
{
  "data": {
    "type": "mfa_setup",
    "attributes": {
      "secret": "JBSWY3DPEHPK3PXP",
      "qr_code_uri": "otpauth://totp/23blocks%20-%20My%20Company:user@example.com?secret=JBSWY3DPEHPK3PXP&issuer=23blocks%20-%20My%20Company",
      "backup_codes": [
        "A1B2C3D4", "E5F6G7H8", "I9J0K1L2",
        "M3N4O5P6", "Q7R8S9T0", "U1V2W3X4",
        "Y5Z6A7B8", "C9D0E1F2", "G3H4I5J6",
        "K7L8M9N0"
      ],
      "test_code": "123456"
    }
  }
}
```

**Errors:**
- `403 Forbidden` - User not authorized (can only setup own MFA)

**Important:** Store the `backup_codes` securely. They are only shown once during setup.

---

### POST /users/:unique_id/mfa/enable - Enable MFA

Enables MFA after verifying a TOTP code. Must call `/mfa/setup` first.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/{unique_id}/mfa/enable" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "totp_code": "123456"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `totp_code` | string | Yes | Current TOTP code from authenticator app |

**Response 200:**
```json
{
  "data": {
    "type": "mfa_status",
    "attributes": {
      "enabled": true,
      "message": "MFA enabled successfully"
    }
  }
}
```

**Errors:**
- `400 Bad Request` - MFA not set up (call `/mfa/setup` first)
- `401 Unauthorized` - Invalid TOTP code
- `403 Forbidden` - User not authorized

---

### POST /users/:unique_id/mfa/disable - Disable MFA

Disables MFA and clears secret and backup codes. Requires current password for security.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/{unique_id}/mfa/disable" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "password": "current_password"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `password` | string | Yes | Current account password |

**Response 200:**
```json
{
  "data": {
    "type": "mfa_status",
    "attributes": {
      "enabled": false,
      "message": "MFA disabled successfully"
    }
  }
}
```

**Errors:**
- `401 Unauthorized` - Invalid password
- `403 Forbidden` - User not authorized

---

### POST /users/:unique_id/mfa/verify - Verify MFA Code

Verifies a TOTP code or backup code. Use during login flow when MFA is enabled.

**Request (TOTP code):**
```bash
curl -X POST "$BLOCKS_API_URL/users/{unique_id}/mfa/verify" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "123456"
  }'
```

**Request (Backup code):**
```bash
curl -X POST "$BLOCKS_API_URL/users/{unique_id}/mfa/verify" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "backup_code": "A1B2C3D4"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Conditional | TOTP code (either `code` or `backup_code` required) |
| `backup_code` | string | Conditional | Backup code (either `code` or `backup_code` required) |

**Response 200:**
```json
{
  "data": {
    "type": "mfa_verification",
    "attributes": {
      "valid": true,
      "message": "MFA verification successful"
    }
  }
}
```

**Errors:**
- `400 Bad Request` - MFA not enabled, or missing both `code` and `backup_code`
- `401 Unauthorized` - Invalid MFA code
- `403 Forbidden` - User not authorized

---

### GET /users/:unique_id/mfa/status - MFA Status

Returns the current MFA status for the user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/{unique_id}/mfa/status" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "mfa_status",
    "attributes": {
      "enabled": true,
      "setup_required": false,
      "backup_codes_remaining": 8,
      "last_used_at": "2026-03-13T10:30:00Z"
    }
  }
}
```

---

## Error Response Format

```json
{
  "errors": [{
    "status": "401",
    "source": "MFA Controller",
    "code": "45365M",
    "title": "Invalid Code",
    "detail": "Invalid TOTP code provided"
  }]
}
```

---

## MFA Setup Flow

```
1. POST /users/:id/mfa/setup     → Get secret, QR URI, backup codes
2. User scans QR code with authenticator app (Google Authenticator, Authy, etc.)
3. POST /users/:id/mfa/enable    → Verify TOTP code to confirm setup
4. MFA is now active for all future logins
```

---

## Important Notes

- **Self-service only**: Users can only manage their own MFA; each endpoint verifies the authenticated user matches the URL's `:unique_id`.
- **TOTP algorithm**: Uses TOTP (RFC 6238) with 30-second drift window (1 period before/after).
- **10 backup codes**: Generated during setup as 8-character alphanumeric codes (uppercase). Each can only be used once.
- **Password required to disable**: Disabling MFA requires the current password for security.
- **Regeneration**: Calling setup with `regenerate: "true"` creates a new secret and resets MFA to disabled.
