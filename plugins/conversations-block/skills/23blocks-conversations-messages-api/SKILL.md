---
name: 23blocks-conversations-messages-api
description: Send, receive, and manage messages within conversations with read tracking and drafts. Use when sending messages, managing read receipts, creating drafts, or updating message content.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Messages API

Send, receive, update, and manage messages within conversations. Supports read/unread tracking, message extensions, and draft messages.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Conversations API base URL | `https://conversations.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

```bash
curl -X GET "$BLOCKS_API_URL/conversations/{unique_id}/messages/{message_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/conversations/:unique_id/messages/:message_id` | Get message |
| POST | `/conversations/:unique_id/messages` | Send message |
| PUT | `/conversations/:unique_id/messages` | Mark message as read (legacy) |
| PUT | `/conversations/:unique_id/messages/:message_unique_id/read` | Mark message as read |
| PUT | `/conversations/:unique_id/messages/:message_unique_id/unread` | Mark message as unread |
| PUT | `/conversations/:unique_id/messages/read_all` | Mark all messages as read |
| PUT | `/conversations/:unique_id/messages/:message_unique_id/extend` | Extend message |
| GET | `/conversations/:unique_id/draft_messages/:message_id` | Get draft message |
| POST | `/conversations/:unique_id/draft_messages` | Create draft message |

---

## Data Model

### Message

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the message |
| conversation_id | string | Parent conversation ID |
| sender_id | string | User who sent the message |
| sender_role | string | Role of the sender (e.g. `user`, `assistant`, `system`) |
| content | string | Message body content |
| message_type | string | Type: `text`, `image`, `file`, `system`, `rich` |
| read_by | array | List of user IDs who have read the message |
| attachments | array | Array of file attachment objects |
| status | string | Delivery status: `sent`, `delivered` |
| edited | boolean | Whether the message has been edited |
| edited_at | datetime | Timestamp of last edit |
| extended_data | object | Custom data attached via /extend |
| metadata | object | Arbitrary key-value metadata |
| expires_at | datetime | Message expiration timestamp (null = never expires) |
| idempotency_key | string | Client-provided key for deduplication |
| rag_sources | object (JSONB) | RAG retrieval source references |
| actions | array | Inline message actions (created with message) |
| created_at | datetime | Message creation timestamp |
| updated_at | datetime | Last update timestamp |

### Draft Message

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the draft |
| conversation_id | string | Parent conversation ID |
| sender_id | string | User who created the draft |
| content | string | Draft message content |
| message_type | string | Type: `text`, `image`, `file`, `rich` |
| attachments | array | Array of draft attachment objects |
| metadata | object | Arbitrary key-value metadata |
| created_at | datetime | Draft creation timestamp |
| updated_at | datetime | Last update timestamp |

### Attachment

| Field | Type | Description |
|-------|------|-------------|
| file_unique_id | string | Unique identifier for the file |
| filename | string | Original filename |
| content_type | string | MIME type |
| size | integer | File size in bytes |
| url | string | CDN URL for file access |

---

## Breaking Changes

> **Message status no longer changes to `'read'`.** Read tracking is now per-user via `MessageReadReceipt` records (see the **23blocks-conversations-read-receipts-api** skill). The `status` field remains `sent` or `delivered`.

## New Features

### Idempotency

Pass an `idempotency_key` when creating a message to prevent duplicates. If a message with the same key already exists, the API returns the existing message with status `200 OK` and header `X-Idempotency-Status: duplicate` instead of creating a new one.

### Message Expiration

Set `expires_at` on a message to make it auto-expire. Expired messages are excluded from conversation queries (`.not_expired` scope). Use the extend endpoint to update expiration.

### Inline Actions

Pass an `actions` array when creating a message to attach interactive controls (buttons, links, inputs). See the **23blocks-conversations-message-actions-api** skill for the full action data model.

### Sender Role

The `sender_role` field identifies the sender's role (e.g., `user`, `assistant`, `system`). Useful for AI/chatbot conversations.

### RAG Sources

The `rag_sources` JSONB field stores retrieval-augmented generation source references for AI-generated messages.

---

## Error Response Format

```json
{
  "errors": [
    {
      "status": "401",
      "title": "Unauthorized",
      "detail": "Invalid or missing authentication token"
    }
  ]
}
```

Common status codes: `401` Unauthorized, `403` Forbidden, `404` Not Found, `422` Unprocessable Entity.

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-conversations
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
// MessagesService — client.conversations.messages
list(params?: ListMessagesParams): Promise<PageResult<Message>>;
get(uniqueId: string): Promise<Message>;
create(data: CreateMessageRequest): Promise<Message>;
update(uniqueId: string, data: UpdateMessageRequest): Promise<Message>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<Message>;
listByContext(contextId: string, params?: ListMessagesParams): Promise<PageResult<Message>>;
listByParent(parentId: string, params?: ListMessagesParams): Promise<PageResult<Message>>;
listDeleted(params?: ListMessagesParams): Promise<PageResult<Message>>;
```

### TypeScript Types

```typescript
import type {
  Message,
  CreateMessageRequest,
  UpdateMessageRequest,
  ListMessagesParams,
} from '@23blocks/block-conversations';
```

### React Hook

```typescript
import { useConversationsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useConversationsBlock();
  const result = await client.conversations.messages.list();
}
```
