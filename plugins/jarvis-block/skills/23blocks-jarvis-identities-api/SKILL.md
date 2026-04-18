---
name: 23blocks-jarvis-identities-api
description: Manage 23blocks Jarvis user identities via REST API. Use when registering users, managing user contexts, handling user conversations and messages, retrieving user content, or managing delegations.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.1"
---

# Identities API

Complete API reference for 23blocks Jarvis user identity management with contexts, conversations, content, and delegations.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Jarvis API base URL | `https://jarvis.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://jarvis.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://jarvis.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/identities` | List all identities |
| GET | `/identities/:id` | Get a single identity |
| POST | `/identities/:id/register` | Register a user |
| PUT | `/identities/:id` | Update an identity |
| GET | `/identities/:id/contexts` | List user contexts |
| POST | `/identities/:id/contexts` | Create user context |
| GET | `/identities/:id/conversations` | List user conversations |
| POST | `/identities/:id/conversations` | Create user conversation |
| GET | `/identities/:id/conversations/:conv_id/messages` | List user messages |
| POST | `/identities/:id/conversations/:conv_id/messages` | Send user message |
| GET | `/identities/:id/content` | Get user content |
| POST | `/identities/:id/delegations` | Create a delegation |
| GET | `/identities/:id/delegations/granted` | List delegations granted by user |
| GET | `/identities/:id/delegations/received` | List delegations received by user |
| GET | `/identities/:id/delegations/:delegation_id` | Get a single delegation |
| DELETE | `/identities/:id/delegations/:delegation_id` | Revoke a delegation |

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

### Delegation
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `grantor_uid` | uuid | User who granted the delegation |
| `delegate_uid` | uuid | User who received the delegation |
| `agent_uid` | uuid | Agent the delegation applies to |
| `context_uid` | uuid | Context the delegation applies to |
| `permissions` | array | Granted permissions: `read`, `write`, `execute` |
| `status` | enum | active, revoked, expired |
| `expires_at` | timestamp | Delegation expiration time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Delegations

### POST /identities/:id/delegations - Create Delegation

Creates a new delegation from this identity to another user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/user-uuid-123/delegations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "delegation": {
      "delegate_uid": "user-uuid-456",
      "agent_uid": "agent-uuid-789",
      "context_uid": "context-uuid-001",
      "permissions": ["read", "write", "execute"],
      "expires_at": "2025-06-01T00:00:00Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `delegate_uid` | uuid | Yes | User UID to delegate to |
| `agent_uid` | uuid | Yes | Agent the delegation applies to |
| `context_uid` | uuid | No | Context scope (omit for agent-wide) |
| `permissions` | array | No | Permissions: `read`, `write`, `execute` |
| `expires_at` | timestamp | No | Delegation expiration time |

**Response 201:**
```json
{
  "data": {
    "id": "delegation-uuid-001",
    "type": "delegation",
    "attributes": {
      "unique_id": "delegation-uuid-001",
      "grantor_uid": "user-uuid-123",
      "delegate_uid": "user-uuid-456",
      "agent_uid": "agent-uuid-789",
      "context_uid": "context-uuid-001",
      "permissions": ["read", "write", "execute"],
      "status": "active",
      "expires_at": "2025-06-01T00:00:00Z",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### GET /identities/:id/delegations/granted - List Granted Delegations

Lists all delegations granted by this identity.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123/delegations/granted" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "delegation-uuid-001",
      "type": "delegation",
      "attributes": {
        "unique_id": "delegation-uuid-001",
        "grantor_uid": "user-uuid-123",
        "delegate_uid": "user-uuid-456",
        "agent_uid": "agent-uuid-789",
        "permissions": ["read", "write", "execute"],
        "status": "active",
        "created_at": "2025-01-12T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /identities/:id/delegations/received - List Received Delegations

Lists all delegations received by this identity.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-456/delegations/received" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "delegation-uuid-001",
      "type": "delegation",
      "attributes": {
        "unique_id": "delegation-uuid-001",
        "grantor_uid": "user-uuid-123",
        "delegate_uid": "user-uuid-456",
        "agent_uid": "agent-uuid-789",
        "permissions": ["read", "write", "execute"],
        "status": "active",
        "created_at": "2025-01-12T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /identities/:id/delegations/:delegation_id - Get Delegation

Retrieves a single delegation by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123/delegations/delegation-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "delegation-uuid-001",
    "type": "delegation",
    "attributes": {
      "unique_id": "delegation-uuid-001",
      "grantor_uid": "user-uuid-123",
      "delegate_uid": "user-uuid-456",
      "agent_uid": "agent-uuid-789",
      "context_uid": "context-uuid-001",
      "permissions": ["read", "write", "execute"],
      "status": "active",
      "expires_at": "2025-06-01T00:00:00Z",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Delegation not found

---

### DELETE /identities/:id/delegations/:delegation_id - Revoke Delegation

Revokes a delegation.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/identities/user-uuid-123/delegations/delegation-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Delegation not found
- `403 Forbidden` - Not the grantor of this delegation

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

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-jarvis
```

### Setup

```typescript
import { create23BlocksClient } from '@23blocks/sdk';

const client = create23BlocksClient({
  authToken: process.env.BLOCKS_AUTH_TOKEN!,
  apiKey: process.env.BLOCKS_API_KEY!,
  apiUrl: process.env.BLOCKS_API_URL!,
});
```

### Available Methods

```typescript
// JarvisUsersService — client.jarvis.users
list(params?: ListJarvisUsersParams): Promise<PageResult<JarvisUser>>;
get(uniqueId: string): Promise<JarvisUser>;
register(uniqueId: string, data?: RegisterJarvisUserRequest): Promise<JarvisUser>;
update(uniqueId: string, data: UpdateJarvisUserRequest): Promise<JarvisUser>;
addPrompt(uniqueId: string, promptUniqueId: string): Promise<void>;
createContext(uniqueId: string, data?: CreateContextRequest): Promise<unknown>;
sendMessage(uniqueId: string, contextUniqueId: string, data: SendMessageRequest): Promise<unknown>;
```

### TypeScript Types

```typescript
import type {
  JarvisUser,
  RegisterJarvisUserRequest,
  UpdateJarvisUserRequest,
  ListJarvisUsersParams,
  CreateContextRequest,
  SendMessageRequest,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();
  const result = await client.jarvis.users.list();
}
```
