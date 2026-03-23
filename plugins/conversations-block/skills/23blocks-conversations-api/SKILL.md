---
name: 23blocks-conversations-api
description: Create and manage conversations with metadata, archiving, and file uploads. Use when initiating user conversations, uploading files, generating presigned URLs, or organizing conversation data.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Conversations API

Create and manage conversations between users. Supports metadata management, archiving, restoring, file uploads, and presigned URLs for direct file uploads.

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


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users/:unique_id/conversations` | List user conversations |
| GET | `/users/:unique_id/mygroups/conversations` | List group conversations |
| GET | `/conversations/:unique_id` | Get conversation |
| POST | `/conversations/` | Create conversation |
| PUT | `/conversations/:unique_id/meta` | Update conversation metadata |
| PUT | `/conversations/:unique_id/archive` | Archive conversation |
| PUT | `/conversations/:unique_id/restore` | Restore conversation |
| PUT | `/conversations/:unique_id/extend` | Extend conversation |
| PUT | `/conversations/:unique_id/messages/read_all` | Mark all messages as read |
| GET | `/conversations/:unique_id/files/:file_unique_id` | Get file |
| PUT | `/conversations/:unique_id/presign` | Presign file upload |
| POST | `/conversations/:unique_id/files` | Upload file |
| DELETE | `/conversations/:unique_id/files/:file_unique_id` | Delete file |

---

## Data Model

### Conversation

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the conversation |
| participants | array | List of user unique_ids in the conversation |
| last_message | object | Summary of the most recent message |
| unread_count | integer | Number of unread messages for the requesting user |
| metadata | object | Arbitrary key-value metadata |
| extended_data | object | Custom extended data attached via /extend |
| group_id | string | Associated group ID (null for direct conversations) |
| first_response_tracking | object | Tracks first response time and metadata |
| status | string | Conversation status: `active`, `archived` |
| created_at | datetime | Conversation creation timestamp |
| updated_at | datetime | Last update timestamp |

### File

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the file |
| conversation_id | string | Parent conversation ID |
| filename | string | Original filename |
| content_type | string | MIME type |
| size | integer | File size in bytes |
| url | string | CDN URL for file access |
| uploaded_by | string | User who uploaded the file |
| metadata | object | File metadata |
| created_at | datetime | Upload timestamp |

---

## Breaking Changes

> **Read status is now per-user via read horizon.** The conversation-level `unread_count` is now computed per-user using individual `MessageReadReceipt` records and a `last_read_at` read horizon on `context_users`. The old behavior where message `status` changed to `'read'` globally is no longer in effect. See the **23blocks-conversations-read-receipts-api** skill for details.

## New Features

### First Response Tracking

The `first_response_tracking` field on contexts tracks when the first response was sent in a conversation, useful for SLA and response time metrics.

### Auto-Read on Show

When a conversation is retrieved via `GET /conversations/:unique_id`, all messages are automatically marked as read for the requesting user via `MessageReadService.mark_conversation_as_read`.

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

Common status codes: `401` Unauthorized, `404` Not Found, `413` Payload Too Large, `422` Unprocessable Entity.

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
// ConversationsService — client.conversations.conversations
get(params: GetConversationParams): Promise<Conversation>;
listContexts(): Promise<string[]>;
deleteContext(context: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Conversation,
  ConversationFile,
  ConversationMeta,
  GetConversationParams,
} from '@23blocks/block-conversations';
```

### React Hook

```typescript
import { useConversationsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useConversationsBlock();
  const result = await client.conversations.conversations.get({ context: 'my-context' });
}
```
