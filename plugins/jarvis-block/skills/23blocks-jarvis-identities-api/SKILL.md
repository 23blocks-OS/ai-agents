---
name: 23blocks-jarvis-identities-api
description: Manage 23blocks Jarvis user identities via REST API. Use when registering users, managing user contexts, handling user conversations and messages, or retrieving user content.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks Jarvis user identity management with contexts, conversations, and content.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

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
