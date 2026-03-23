---
name: 23blocks-conversations-read-receipts-api
description: Track per-user read receipts for conversation messages with read horizon support. Use when marking messages as read or unread, bulk-reading all messages, or tracking who has read a message.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Read Receipts API

Per-user read receipt tracking for conversation messages. Read status is tracked individually per user via `MessageReadReceipt` records and a read horizon (`last_read_at`) on context_users.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Conversations API base URL | `https://realtime.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://realtime.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://realtime.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Breaking Change

> **Read status is now per-user.** Message `status` field no longer changes to `'read'`. Instead, read receipts are tracked individually via `MessageReadReceipt` records. The presence of a receipt row means the user has read the message; absence means unread. Deleting the row marks it as unread.

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
