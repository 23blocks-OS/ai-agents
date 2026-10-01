---
name: 23blocks-jarvis-conversations-api
description: "Jarvis standalone conversations: messages, AI query, rename, archive. Use for chats not tied to an agent or entity."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Conversations API

Complete API reference for 23blocks Jarvis standalone conversation management with messages, AI queries, and archiving.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://jarvis.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Prerequisites

**User identity must be registered** before calling any endpoint in this skill. Without registration, all requests return `404` with code `usr-not-registered`.

```bash
curl -X POST "$BLOCKS_API_URL/identities/$USER_UNIQUE_ID/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "name": "Your Name", "email": "you@example.com" }'
```

> Self-registration (your own JWT) requires no special scope. Registering other users requires `identities:write`.

---

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

### Error Codes

| Code | Description |
|------|-------------|
| `rt-ctx-get-fail` | GET context failed |
| `rt-ctx-load-503` | Conversations API unreachable (503) |
| `rt-ctx-create-fail` | POST context creation failed |

> **Note:** The previous generic error code `rt-125653` has been split into the three specific codes above for improved diagnostics.

---

## Context Creation Behavior

When creating contexts (for conversations or agents), if no `members` array is provided, Jarvis auto-populates it from the JWT token (user_unique_id + user_email).

> Context `unique_id` must be a valid UUID; other values return 400.

---

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useJarvisBlock()` from `@23blocks/react`.

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
