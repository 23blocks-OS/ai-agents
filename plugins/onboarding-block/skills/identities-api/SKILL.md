---
name: onboarding-identities-api
description: Manage 23blocks Onboarding user identities via REST API. Use when registering users, starting onboardings upon registration, managing user profiles, viewing user journeys, or listing all journeys.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Identities API

Complete API reference for 23blocks Onboarding user identity management with journey tracking and registration.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://onboarding.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Onboarding API base URL | `https://onboarding.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /users/ - List Identities

Lists all user identities in the onboarding system with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

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
        "display_name": "John Doe",
        "status": "active",
        "journeys_count": 3,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### GET /users/:unique_id/ - Get Identity

Retrieves a single user identity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123" \
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
      "display_name": "John Doe",
      "status": "active",
      "journeys_count": 3,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### POST /users/:unique_id/register/ - Register Identity

Registers a new user identity in the onboarding system.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "display_name": "New User"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email |
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
      "display_name": "New User",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### POST /users/:unique_id/register/:onboarding_id - Register and Start Onboarding

Registers a new user identity and automatically starts the specified onboarding journey.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/register/onboarding-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "display_name": "New User"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email |
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
      "display_name": "New User",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    },
    "relationships": {
      "journey": {
        "data": {
          "id": "journey-uuid-789",
          "type": "journey"
        }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Onboarding not found
- `422 Unprocessable Entity` - Validation errors

---

### PUT /users/:unique_id/ - Update Identity

Updates an existing user identity.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "display_name": "John D.",
      "metadata": {"preferred_language": "en"}
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
      "metadata": {"preferred_language": "en"},
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### GET /users/:unique_id/journey - Get User Journey (Admin)

Retrieves the current active journey for a user. Admin endpoint.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/journey" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "journey-uuid-789",
    "type": "journey",
    "attributes": {
      "unique_id": "journey-uuid-789",
      "onboarding_id": "onboarding-uuid-456",
      "user_unique_id": "user-uuid-123",
      "current_step": 2,
      "status": "active",
      "started_at": "2025-01-10T10:30:00Z",
      "completed_at": null
    }
  }
}
```

**Errors:**
- `404 Not Found` - User or journey not found

---

### GET /users/:unique_id/journeys - Get All User Journeys

Retrieves all journeys for a specific user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/journeys?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "journey-uuid-789",
      "type": "journey",
      "attributes": {
        "unique_id": "journey-uuid-789",
        "onboarding_id": "onboarding-uuid-456",
        "user_unique_id": "user-uuid-123",
        "current_step": 3,
        "status": "completed",
        "started_at": "2025-01-10T10:30:00Z",
        "completed_at": "2025-01-11T15:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 3
  }
}
```

---

### GET /journeys/ - Get All Journeys

Lists all journeys across all users with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/journeys?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "journey-uuid-789",
      "type": "journey",
      "attributes": {
        "unique_id": "journey-uuid-789",
        "onboarding_id": "onboarding-uuid-456",
        "user_unique_id": "user-uuid-123",
        "current_step": 2,
        "status": "active",
        "started_at": "2025-01-10T10:30:00Z",
        "completed_at": null
      }
    }
  ],
  "meta": {
    "totalPages": 10,
    "totalRecords": 150
  }
}
```

---

## Data Models

### UserIdentity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email |
| `display_name` | string | Display name |
| `status` | enum | active, inactive |
| `metadata` | object | Custom metadata |
| `journeys_count` | integer | Number of journeys |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Journey (summary)
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `onboarding_id` | uuid | Associated onboarding |
| `user_unique_id` | uuid | Associated user |
| `current_step` | integer | Current step number |
| `status` | enum | active, completed, suspended |
| `started_at` | timestamp | Journey start time |
| `completed_at` | timestamp | Journey completion time |

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
