---
name: crm-identities-api
description: Manage 23blocks CRM user identities via REST API. Use when registering users, retrieving user profiles, or accessing user contacts and meetings in the CRM system.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# CRM Identities API

Complete API reference for 23blocks CRM user identity management, registration, and associated data retrieval.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://crm.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
echo "All credentials configured"
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | CRM API base URL | `https://crm.api.us.23blocks.com` |
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

### GET /users - List Users

Lists all user identities in the CRM system.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "user-uuid-123",
      "type": "user",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "user@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "role": "agent",
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 5
  }
}
```

---

### GET /users/:unique_id - Get User

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
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "user@example.com",
      "first_name": "John",
      "last_name": "Doe",
      "role": "agent",
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### POST /users/:unique_id/register - Register User

Registers a new user in the CRM system. Each block is autonomous; users must register their identity before using private endpoints.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "first_name": "Jane",
      "last_name": "Smith"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email address |
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |

**Response 201:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "newuser@example.com",
      "first_name": "Jane",
      "last_name": "Smith",
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (e.g., email already registered)

---

### DELETE /users/:unique_id - Delete User

Deletes a user identity from the CRM system.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - User not found

---

### GET /users/:unique_id/contacts - User Contacts

Retrieves all contacts associated with a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/contacts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "contact-uuid-456",
      "type": "contact",
      "attributes": {
        "unique_id": "contact-uuid-456",
        "first_name": "Alice",
        "last_name": "Johnson",
        "email": "alice@example.com",
        "phone": "+1-555-0101",
        "title": "CEO",
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalRecords": 12
  }
}
```

---

### GET /users/:unique_id/meetings - User Meetings

Retrieves all meetings associated with a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/meetings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "meeting-uuid-789",
      "type": "meeting",
      "attributes": {
        "unique_id": "meeting-uuid-789",
        "title": "Weekly Standup",
        "start_time": "2026-02-20T14:00:00Z",
        "end_time": "2026-02-20T14:30:00Z",
        "location": "Conference Room A",
        "meeting_type": "in_person",
        "status": "scheduled",
        "created_at": "2026-01-15T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalRecords": 8
  }
}
```

---

## Data Models

### User
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email address |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `role` | string | User role (e.g., agent, admin) |
| `status` | string | active, inactive |
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
