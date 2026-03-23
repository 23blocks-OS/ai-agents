---
name: 23blocks-auth-magic-links-api
description: Manage 23blocks magic links via REST API. Use when creating, validating, or managing magic links for passwordless authentication. Supports single-step (direct JWT) and two-step (OTP verification) flows.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Magic Links API

Complete API reference for 23blocks magic link management including creation, validation, OTP verification, access logs, and deletion.

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

### POST /companies/:url_id/magic_links - Create Magic Link

Creates a new magic link for passwordless authentication.

**Required Scope:** `magic_links:create`

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/{url_id}/magic_links" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "magic_link": {
      "target_url": "https://app.example.com/dashboard",
      "expired_url": "https://app.example.com/link-expired",
      "user_name": "John Doe",
      "user_unique_id": "user-uuid-123",
      "user_email": "user@example.com",
      "role_id": 2,
      "expires_at": "2026-03-20T00:00:00Z",
      "validation_url": "https://app.example.com/verify",
      "description": "Dashboard access link"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `target_url` | string | Yes | URL to redirect after successful validation |
| `expired_url` | string | No | URL to redirect if link is expired |
| `user_name` | string | No | User display name |
| `user_unique_id` | string | No | User unique ID |
| `user_email` | string | No | User email |
| `role_id` | integer | No | Role ID for the magic link session |
| `expires_at` | datetime | No | Link expiration (max 7 days) |
| `status` | string | No | Initial status |
| `validation_url` | string | No | If set, enables two-step OTP mode |
| `description` | string | No | Link description |
| `image_url` | string | No | Image URL for link metadata |
| `content_url` | string | No | Content URL |
| `media_url` | string | No | Media URL |
| `thumbnail_url` | string | No | Thumbnail URL |

**Response 201:**
```json
{
  "data": {
    "id": "link-uuid-123",
    "type": "magic_link",
    "attributes": {
      "token": "abc123token",
      "target_url": "https://app.example.com/dashboard",
      "status": "new",
      "expires_at": "2026-03-20T00:00:00Z",
      "user_email": "user@example.com"
    }
  }
}
```

---

### GET /companies/:url_id/magic_links/validate - Validate Magic Link

Validates a magic link token. Operates in two modes based on `validation_url` presence:

- **Mode A (Single-step)**: No `validation_url` set. Validates token, issues JWT, redirects to `target_url`.
- **Mode B (Two-step)**: Has `validation_url`. Validates token, generates 6-digit OTP, emails it, returns JSON.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/{url_id}/magic_links/validate?token=abc123token" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | string | Yes | Magic link token |

**Response 200 (Two-step mode):**
```json
{
  "status": "otp_sent",
  "message": "A verification code has been sent to your email.",
  "email_hint": "u***@example.com",
  "validation_url": "https://app.example.com/verify"
}
```

**Response 302 (Single-step mode):**
Redirects to: `{target_url}?company_token={jwt}&appid={url_id}&user_unique_id={uuid}`

**Errors:**
- `400 Bad Request` (EXPIRED) - Link has expired
- `400 Bad Request` (INVALID_TOKEN) - Token not found or invalid
- `400 Bad Request` (INVALID_COMPANY) - Company not found

---

### POST /companies/:url_id/magic_links/:token/verify_code - Verify OTP Code

Step 2 of two-step mode: verifies the OTP code sent via email and returns a JWT.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/{url_id}/magic_links/{token}/verify_code" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "123456"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | 6-digit OTP code received via email |

**Response 200:**
```json
{
  "company_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "appid": "my-company",
  "user_unique_id": "user-uuid-123",
  "target_url": "https://app.example.com/dashboard"
}
```

**Errors:**
- `400 Bad Request` (INVALID_TOKEN) - Token not found or not in `otp_sent` state
- `400 Bad Request` (CODE_EXPIRED) - OTP code expired (10 min TTL)
- `400 Bad Request` (INVALID_CODE) - Wrong OTP code

---

### PUT /companies/:url_id/magic_links/:token/send - Send Magic Link Email

Sends or re-sends the magic link via email.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/companies/{url_id}/magic_links/{token}/send" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### GET /companies/:url_id/magic_links/:token/logs - Access Logs

Lists access logs for a specific magic link with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/companies/{url_id}/magic_links/{token}/logs?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number |
| `records` | integer | No | Records per page |
| `ip` | string | No | Filter by IP address |
| `email` | string | No | Filter by email |
| `name` | string | No | Filter by name |

**Response 200:**
```json
{
  "data": [
    {
      "id": "log-uuid-123",
      "type": "magic_link_log",
      "attributes": {
        "user_agent": "Mozilla/5.0...",
        "user_ip": "192.168.1.1",
        "created_at": "2026-03-13T10:30:00Z"
      }
    }
  ]
}
```

---

### DELETE /companies/:url_id/magic_links/:token - Delete Magic Link

Disables and marks a magic link as deleted.

**Required Scope:** `magic_links:delete`

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/companies/{url_id}/magic_links/{token}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Error Response Format

```json
{
  "errors": [{
    "status": "400",
    "source": "Magic Links - AUTH API",
    "code": "EXPIRED",
    "title": "Magic Link Error",
    "detail": "This magic link has expired."
  }]
}
```

---

## Important Notes

- **Two validation modes**: Set `validation_url` on creation for two-step OTP mode; omit for single-step redirect mode.
- **OTP expiry**: Two-step OTP codes expire after 10 minutes.
- **SHA256 hashing**: OTP codes are stored as SHA256 digests, verified with timing-safe comparison.
- **JWT issued on success**: Both modes issue a 24-hour JWT with the user's role permissions as scopes.
- **Max 7-day expiry**: Magic links can be set to expire up to 7 days from creation.
