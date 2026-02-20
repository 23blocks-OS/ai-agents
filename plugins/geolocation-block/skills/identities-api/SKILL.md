---
name: geolocation-identities-api
description: Manage 23blocks geolocation user identities via REST API. Use when registering users, managing user profiles, or setting user locations within the geolocation system.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Identities API

Complete API reference for 23blocks geolocation user identity management and user location tracking.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://geolocation.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Geolocation API base URL | `https://geolocation.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/identities/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /identities/ - List Identities

Lists all user identities registered in the geolocation system.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "user-uuid-123",
      "type": "user_identity",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "user@example.com",
        "username": "johndoe",
        "display_name": "John Doe",
        "avatar_url": "https://example.com/avatar.jpg",
        "latitude": 40.7128,
        "longitude": -74.0060,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /identities/:unique_id/ - Get Identity

Retrieves a single user identity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123/" \
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
      "email": "user@example.com",
      "username": "johndoe",
      "display_name": "John Doe",
      "avatar_url": "https://example.com/avatar.jpg",
      "latitude": 40.7128,
      "longitude": -74.0060,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### POST /identities/:unique_id/register/ - Register Identity

Registers a new user in the geolocation system. Each block is autonomous; users must register their identity before using private endpoints.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/user-uuid-123/register/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "username": "newuser",
      "display_name": "New User"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email |
| `username` | string | No | Username |
| `display_name` | string | No | Display name |

**Response 201:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "newuser@example.com",
      "username": "newuser",
      "display_name": "New User",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - User already registered
- `422 Unprocessable Entity` - Validation errors

---

### PUT /identities/:unique_id/ - Update Identity

Updates an existing user profile.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/identities/user-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "display_name": "John D.",
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
      "display_name": "John D.",
      "avatar_url": "https://example.com/new-avatar.jpg",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### DELETE /identities/:unique_id/ - Delete Identity

Deletes a user identity from the geolocation system.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/identities/user-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - User not found

---

## User Location

### POST /users/:unique_id/location - Set User Location

Sets or updates the current geographic location for a user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/location" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "location": {
      "latitude": 40.7128,
      "longitude": -74.0060
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `latitude` | float | Yes | Latitude coordinate |
| `longitude` | float | Yes | Longitude coordinate |

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "latitude": 40.7128,
      "longitude": -74.0060,
      "location_updated_at": "2025-01-12T14:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found
- `422 Unprocessable Entity` - Invalid coordinates

---

## Data Models

### UserIdentity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email |
| `username` | string | Username |
| `display_name` | string | Display name |
| `avatar_url` | string | Avatar image URL |
| `latitude` | float | Current latitude |
| `longitude` | float | Current longitude |
| `location_updated_at` | timestamp | Last location update time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "User Not Found",
    "detail": "The requested user could not be found."
  }]
}
```
