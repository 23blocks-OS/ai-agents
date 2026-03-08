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
| `BLOCKS_API_URL` | Conversations API base URL | `https://conversations.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

```bash
curl -X GET "$BLOCKS_API_URL/users/{unique_id}/conversations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
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
