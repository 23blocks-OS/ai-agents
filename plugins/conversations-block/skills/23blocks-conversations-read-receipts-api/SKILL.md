---
name: 23blocks-conversations-read-receipts-api
description: "Conversations Block per-user read state: read, unread, read all, who read a message. Use for unread counts and receipts."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Read Receipts API

Per-user read receipt tracking for conversation messages. Read status is tracked individually per user via `MessageReadReceipt` records and a read horizon (`last_read_at`) on context_users.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://realtime.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## How read state works

Read state is per user. Message `status` never becomes `'read'`; each user's reads are `MessageReadReceipt` records. The presence of a receipt row means the user has read the message; absence means unread. Deleting the row marks it as unread.

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| PUT | `/conversations/:unique_id/messages/:message_unique_id/read` | Mark a message as read for the current user |
| PUT | `/conversations/:unique_id/messages/:message_unique_id/unread` | Mark a message as unread for the current user |
| PUT | `/conversations/:unique_id/messages/read_all` | Mark all messages in a conversation as read |

> **Auto-read on show:** When a conversation is retrieved via `GET /conversations/:unique_id`, all messages are automatically marked as read for the requesting user.

---

## How It Works

1. **Mark as read** creates a `MessageReadReceipt` row linking the user to the message with a `read_at` timestamp.
2. **Mark as unread** removes the user's `MessageReadReceipt` row for that message.
3. **Read all** marks all messages in the conversation as read and updates the user's `last_read_at` read horizon on `context_users`.
4. **Auto-read on show** — viewing a conversation via GET automatically marks it as read for the requesting user.

---

## Data Model

### MessageReadReceipt

| Field | Type | Description |
|-------|------|-------------|
| message_unique_id | string | The message that was read |
| context_unique_id | string | The conversation context |
| user_unique_id | string | The user who read the message |
| user_name | string | Display name of the reader |
| role_name | string | Role of the reader |
| read_at | datetime | When the message was read |

**Uniqueness constraint:** One receipt per user per message (`user_unique_id` scoped to `message_id`).

---

## WebSocket Events

Read receipts broadcast to `conversation_{context_unique_id}` channel on creation with `object_type: 'read_receipt'`.

**Payload:**

```json
{
  "message_unique_id": "msg_001",
  "context_unique_id": "conv_def456",
  "user_unique_id": "usr_xyz789",
  "user_name": "Jane Smith",
  "role_name": "member",
  "read_at": "2025-01-15T11:00:00Z",
  "object_type": "read_receipt"
}
```

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
