---
name: search-identities-api
description: Manage 23blocks Search user identities via REST API. Use when registering users in the search block, retrieving identity details, or updating user profiles.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Identities API

Complete API reference for 23blocks Search identity registration and management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://search.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Search API base URL | `https://search.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /identities/:unique_id - Get Identity

Retrieves a user identity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "user_unique_id": "user-uuid-123",
      "name": "John Doe",
      "email": "user@example.com",
      "phone": "+1234567890",
      "avatar_url": "https://example.com/avatar.jpg",
      "role_id": 4,
      "role_name": "user",
      "status": "active",
      "enabled": true,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Identity not found

---

### POST /identities/:unique_id/register - Register Identity

Registers a new user identity in the search block.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/user-uuid-123/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "name": "New User",
      "phone": "+1234567890"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email address |
| `name` | string | No | User display name |
| `phone` | string | No | Phone number |

**Response 201:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "user_unique_id": "user-uuid-123",
      "email": "newuser@example.com",
      "name": "New User",
      "role_id": 4,
      "role_name": "user",
      "status": "active",
      "enabled": true,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Identity already registered
- `422 Unprocessable Entity` - Validation errors

---

### PUT /identities/:unique_id - Update Identity

Updates an existing user identity.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/identities/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "name": "John Updated",
      "avatar_url": "https://example.com/new-avatar.jpg"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "name": "John Updated",
      "avatar_url": "https://example.com/new-avatar.jpg",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Data Models

### UserIdentity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | User unique ID from auth provider |
| `name` | string | Display name |
| `email` | string | Email address |
| `phone` | string | Phone number |
| `avatar_url` | string | Avatar image URL |
| `role_id` | integer | Role ID (1=admin, 2=manager, 4=user) |
| `role_name` | string | Role name |
| `status` | string | Account status (active, inactive) |
| `enabled` | boolean | Whether user is enabled |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Identity Not Found",
    "detail": "The requested identity could not be found."
  }]
}
```
