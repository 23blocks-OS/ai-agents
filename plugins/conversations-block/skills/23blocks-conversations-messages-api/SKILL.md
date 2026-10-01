---
name: 23blocks-conversations-messages-api
description: "Conversations Block messages: send, update, extend, drafts, idempotency. Use for message content; read state is 23blocks-conversations-read-receipts-api."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Messages API

Send, receive, update, and manage messages within conversations. Supports read/unread tracking, message extensions, and draft messages.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://realtime.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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
| idempotency_key | string | Client-provided deduplication key (`[A-Za-z0-9:._-]`, 20-128 chars) |
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

## Behaviour notes

Message `status` is only ever `sent` or `delivered`. Read state is per user, in `MessageReadReceipt` records (see the **23blocks-conversations-read-receipts-api** skill).

### Idempotency

Pass an `idempotency_key` when creating a message to prevent duplicates. If a message with the same key was created within the last 72 hours, the API returns the original message with status `200 OK` and header `X-Idempotency-Status: duplicate` instead of creating a new one.

- Accepted format: `[A-Za-z0-9:._-]`, 20-128 chars. Composite/namespaced keys like `<event>:<uuid>` are valid without sanitizing.
- Dedup is keyed only on `idempotency_key`. The API does not dedup on message content — identical bodies without a key are distinct messages.
- Recommendation: send a stable `idempotency_key` per logical action for retry-safety.

### Event Name

Pass an optional `event_name` (string) when creating a message. It identifies the business event (e.g. `pcu_test_created`) and selects which notification template renders the resulting email/SMS. The mailer prefers `event_name` and falls back to `source_type` when omitted.

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

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-conversations`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useConversationsBlock()` from `@23blocks/react`.

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
