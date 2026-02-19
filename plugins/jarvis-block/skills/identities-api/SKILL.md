---
name: jarvis-identities-api
description: Manage 23blocks Jarvis user identities via REST API. Use when registering users, managing user contexts, handling user conversations and messages, or retrieving user content.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Identities API

Complete API reference for 23blocks Jarvis user identity management with contexts, conversations, and content.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://jarvis.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Jarvis API base URL | `https://jarvis.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/identities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /identities - List Identities

Lists all user identities with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities?page=1&records=20" \
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
      "id": "identity-uuid-123",
      "type": "identity",
      "attributes": {
        "unique_id": "identity-uuid-123",
        "email": "user@example.com",
        "display_name": "John Doe",
        "status": "active",
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

### GET /identities/:id - Get Identity

Retrieves a single identity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/identity-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "identity-uuid-123",
    "type": "identity",
    "attributes": {
      "unique_id": "identity-uuid-123",
      "email": "user@example.com",
      "display_name": "John Doe",
      "status": "active",
      "contexts_count": 3,
      "conversations_count": 12,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Identity not found

---

### POST /identities/:id/register - Register User

Registers a new user identity in the Jarvis system.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/identity-uuid-123/register" \
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
    "id": "identity-uuid-123",
    "type": "identity",
    "attributes": {
      "unique_id": "identity-uuid-123",
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

### PUT /identities/:id - Update Identity

Updates an existing user identity.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/identities/identity-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "display_name": "John D.",
      "metadata": {"role": "admin"}
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "identity-uuid-123",
    "type": "identity",
    "attributes": {
      "unique_id": "identity-uuid-123",
      "display_name": "John D.",
      "metadata": {"role": "admin"},
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## User Contexts

### GET /identities/:id/contexts - List User Contexts

Retrieves all contexts for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/identity-uuid-123/contexts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "context-uuid-456",
      "type": "context",
      "attributes": {
        "unique_id": "context-uuid-456",
        "name": "Work Profile",
        "content": "User preferences and settings for work context",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /identities/:id/contexts - Create User Context

Creates a new context for a user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/identity-uuid-123/contexts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "context": {
      "name": "Work Profile",
      "content": "User is a senior engineer who prefers concise responses."
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Context name |
| `content` | string | Yes | Context content/instructions |

**Response 201:**
```json
{
  "data": {
    "id": "context-uuid-456",
    "type": "context",
    "attributes": {
      "unique_id": "context-uuid-456",
      "name": "Work Profile",
      "content": "User is a senior engineer who prefers concise responses.",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## User Conversations

### GET /identities/:id/conversations - List User Conversations

Lists all conversations for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/identity-uuid-123/conversations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "conv-uuid-789",
      "type": "conversation",
      "attributes": {
        "unique_id": "conv-uuid-789",
        "title": "Project Discussion",
        "messages_count": 15,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 25
  }
}
```

---

### POST /identities/:id/conversations - Create User Conversation

Creates a new conversation for a user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/identity-uuid-123/conversations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation": {
      "title": "Project Discussion"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "conv-uuid-789",
    "type": "conversation",
    "attributes": {
      "unique_id": "conv-uuid-789",
      "title": "Project Discussion",
      "messages_count": 0,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### GET /identities/:id/conversations/:conv_id/messages - List User Messages

Lists messages in a user conversation.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/identity-uuid-123/conversations/conv-uuid-789/messages?page=1&records=50" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "msg-uuid-101",
      "type": "message",
      "attributes": {
        "unique_id": "msg-uuid-101",
        "role": "user",
        "content": "What is the project status?",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "msg-uuid-102",
      "type": "message",
      "attributes": {
        "unique_id": "msg-uuid-102",
        "role": "assistant",
        "content": "The project is on track for the Q1 deadline.",
        "created_at": "2025-01-10T10:30:05Z"
      }
    }
  ]
}
```

---

### POST /identities/:id/conversations/:conv_id/messages - Send User Message

Sends a message in a user conversation.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/identity-uuid-123/conversations/conv-uuid-789/messages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "role": "user",
      "content": "What are the next milestones?"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `role` | string | Yes | Message role (user, assistant, system) |
| `content` | string | Yes | Message content |

**Response 201:**
```json
{
  "data": {
    "id": "msg-uuid-103",
    "type": "message",
    "attributes": {
      "unique_id": "msg-uuid-103",
      "role": "user",
      "content": "What are the next milestones?",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## User Content

### GET /identities/:id/content - Get User Content

Retrieves content associated with a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/identity-uuid-123/content" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "content-uuid",
      "type": "content",
      "attributes": {
        "unique_id": "content-uuid",
        "title": "User Notes",
        "content_type": "note",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

## Data Models

### Identity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email |
| `display_name` | string | Display name |
| `status` | enum | active, inactive |
| `metadata` | object | Custom metadata |
| `contexts_count` | integer | Number of contexts |
| `conversations_count` | integer | Number of conversations |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Context
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Context name |
| `content` | string | Context content |
| `created_at` | timestamp | Creation time |

### Message
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `role` | enum | user, assistant, system |
| `content` | string | Message content |
| `created_at` | timestamp | Creation time |

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
