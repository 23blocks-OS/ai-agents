---
name: 23blocks-auth-guests-api
description: Manage 23blocks guest users via REST API. Use when creating guest sessions, updating guest information, converting guests to registered users, or re-authenticating expired guest tokens.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Guests API

Complete API reference for 23blocks guest user lifecycle management including creation, updates, conversion to registered users, and re-authentication.

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

### POST /guests/ - Create Guest

Creates a new guest user or returns an existing one (matched by unique_id or email). Issues a 24-hour JWT token.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/guests/" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "guest": {
      "email": "visitor@example.com",
      "name": "Anonymous Visitor",
      "username": "visitor123"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `guest.unique_id` | string | No | Guest unique ID (for matching existing guests) |
| `guest.email` | string | No | Guest email (for matching existing guests) |
| `guest.username` | string | No | Guest username |
| `guest.name` | string | No | Guest display name |
| `guest.status` | string | No | Guest status |
| `guest.user_unique_id` | string | No | Associated user unique ID |

**Response 201:**
```json
{
  "data": {
    "id": "guest-uuid-123",
    "type": "guest",
    "attributes": {
      "unique_id": "guest-uuid-123",
      "email": "visitor@example.com",
      "name": "Anonymous Visitor",
      "username": "visitor123",
      "status": "active",
      "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
      "current_visiting_ip": "192.168.1.1",
      "current_visiting_at": "2026-03-13T10:30:00Z"
    }
  }
}
```

**Response Headers:**
- `company-token`: JWT token
- `authorization`: `bearer {JWT}`

**Notes:**
- If a guest with the same `unique_id` or `email` already exists, it returns the existing guest (idempotent).
- Supports JSON payload extension via the `payload` field.

---

### GET /guests/:user_unique_id - Show Guest

Retrieves a guest user by their user unique ID. Updates visit tracking (IP and timestamp).

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/guests/{user_unique_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "guest-uuid-123",
    "type": "guest",
    "attributes": {
      "unique_id": "guest-uuid-123",
      "email": "visitor@example.com",
      "name": "Anonymous Visitor",
      "status": "active",
      "current_visiting_ip": "192.168.1.1",
      "current_visiting_at": "2026-03-13T10:30:00Z",
      "last_visiting_ip": "192.168.1.1",
      "last_visiting_at": "2026-03-12T15:00:00Z"
    }
  }
}
```

**Errors:**
- `400 Bad Request` - Missing or invalid user unique ID format (must be UUID)
- `404 Not Found` - Guest not found

---

### PUT /guests/:user_unique_id - Update Guest

Updates guest information and issues a new JWT token.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/guests/{user_unique_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "guest": {
      "email": "updated@example.com",
      "name": "Updated Name"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "guest-uuid-123",
    "type": "guest",
    "attributes": {
      "unique_id": "guest-uuid-123",
      "email": "updated@example.com",
      "name": "Updated Name",
      "status": "active"
    }
  }
}
```

**Response Headers:**
- `company-token`: New JWT token

---

### PUT /guests/:user_unique_id/convert - Convert Guest to User

Converts a guest to a registered user by updating their identity and marking them as converted.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/guests/{user_unique_id}/convert" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "guest": {
      "user_unique_id": "registered-user-uuid",
      "email": "registered@example.com",
      "username": "registereduser",
      "name": "Registered User"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `guest.user_unique_id` | string | No | Registered user unique ID |
| `guest.email` | string | No | Registered user email |
| `guest.username` | string | No | Registered username |
| `guest.name` | string | No | Registered display name |

**Response 200:**
```json
{
  "data": {
    "id": "guest-uuid-123",
    "type": "guest",
    "attributes": {
      "unique_id": "guest-uuid-123",
      "status": "converted",
      "registered_at": "2026-03-13T10:30:00Z",
      "user_unique_id": "registered-user-uuid"
    }
  }
}
```

---

### POST /guests/:user_unique_id/auth - Re-authenticate Guest

Re-authenticates a guest and issues a new 24-hour JWT token. Use when the previous guest token has expired.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/guests/{user_unique_id}/auth" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "guest-uuid-123",
    "type": "guest",
    "attributes": {
      "unique_id": "guest-uuid-123",
      "email": "visitor@example.com",
      "name": "Anonymous Visitor",
      "status": "active"
    }
  }
}
```

**Response Headers:**
- `company-token`: New JWT token

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Not Found",
    "detail": "Guest with uniqueId abc-123 not Found"
  }]
}
```

---

## Important Notes

- **Visit tracking**: Every `show`, `create`, `update`, `convert`, and `authenticate` call updates the guest's visit IP and timestamp.
- **Idempotent creation**: Creating a guest with an existing `unique_id` or `email` returns the existing guest.
- **UUID validation**: The `user_unique_id` path parameter must be a valid UUID format.
- **24-hour tokens**: Guest JWT tokens expire after 24 hours. Use the `/auth` endpoint to re-authenticate.
- **Guest role**: Guests are assigned the guest role (ID: 1 or code: `guest`) with associated permissions.
- **Payload support**: Custom JSON payloads can be attached to guests via the `payload` field.
