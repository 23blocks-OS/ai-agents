---
name: 23blocks-jarvis-conversations-api
description: Manage 23blocks Jarvis standalone conversations via REST API. Use when creating conversations, sending messages, querying conversations with AI, or archiving and restoring conversations.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Conversations API

Complete API reference for 23blocks Jarvis standalone conversation management with messages, AI queries, and archiving.

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
| GET | `/conversations` | List all conversations |
| GET | `/conversations/:id` | Get a conversation |
| POST | `/conversations` | Create a conversation |
| PUT | `/conversations/:id` | Update a conversation |
| DELETE | `/conversations/:id` | Delete a conversation |
| GET | `/conversations/:id/messages` | List messages |
| POST | `/conversations/:id/messages` | Send a message |
| POST | `/conversations/:id/query` | Query conversation with AI |
| PUT | `/conversations/:id/rename` | Rename a conversation |
| PUT | `/conversations/:id/archive` | Archive a conversation |
| PUT | `/conversations/:id/restore` | Restore a conversation |

---

## Data Models

### Conversation
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Conversation title |
| `messages_count` | integer | Number of messages |
| `status` | enum | active, archived |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

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
    "title": "Conversation Not Found",
    "detail": "The requested conversation could not be found."
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
// ConversationsService — client.jarvis.conversations
list(params?: ListConversationsParams): Promise<PageResult<Conversation>>;
get(uniqueId: string): Promise<Conversation>;
create(data: CreateConversationRequest): Promise<Conversation>;
sendMessage(uniqueId: string, data: SendConversationMessageRequest): Promise<SendConversationMessageResponse>;
listByUser(userUniqueId: string, params?: ListConversationsParams): Promise<PageResult<Conversation>>;
clear(uniqueId: string): Promise<Conversation>;
```

### TypeScript Types

```typescript
import type {
  Conversation,
  ConversationMessage,
  CreateConversationRequest,
  SendConversationMessageRequest,
  SendConversationMessageResponse,
  ListConversationsParams,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();
  const result = await client.jarvis.conversations.list();
}
```
