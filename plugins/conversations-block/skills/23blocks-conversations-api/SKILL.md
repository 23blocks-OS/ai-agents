---
name: 23blocks-conversations-api
description: Create and manage conversations with metadata, archiving, file uploads, AI summaries, and task management. Use when initiating user conversations, uploading files, generating presigned URLs, organizing conversation data, generating AI-powered conversation summaries, or managing conversation tasks.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.5"
---

# Conversations API

Create and manage conversations between users. Supports metadata management, archiving, restoring, file uploads, presigned URLs for direct file uploads, AI-powered summaries, and task management.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Conversations API base URL | `https://realtime.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

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
| GET | `/users/:unique_id/conversations` | List user conversations (sortable, filterable) |
| GET | `/users/:unique_id/mygroups/conversations` | List group conversations (searchable, sortable) |
| GET | `/users/:unique_id/unread-summary` | Aggregated unread counts by dimension (supports payload grouping) |
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
| GET | `/conversations/:unique_id/summary` | Get AI summary (via Jarvis) |
| GET | `/users/:unique_id/conversations/summary` | Get user conversations digest |
| GET | `/conversations/:context_id/tasks` | List tasks for a conversation |
| GET | `/users/:user_id/tasks` | Task digest for a user (all conversations) |
| POST | `/conversations/:context_id/tasks` | Create manual task |
| PUT | `/tasks/:unique_id` | Update task (body or `?action_type=complete\|dismiss\|reopen`) |
| DELETE | `/tasks/:unique_id` | Delete task |

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

### ConversationSummary

> **Relationship:** Conversations have an optional `summary` (has_one) relationship. Include via `?include=summary`. User-scoped: each user gets their own summary perspective.

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the summary |
| conversation_id | string | Parent conversation ID |
| overall_summary | text | AI-generated conversation summary |
| action_items | jsonb | Extracted action items |
| key_points | jsonb | Key discussion points extracted from conversation |
| key_decisions | jsonb | Extracted key decisions |
| important_info | jsonb | Important information extracted |
| participants | jsonb | Participant details from summary |
| sentiment | string | Overall sentiment: `positive`, `neutral`, `negative`, `mixed` |
| conversation_status | string | Status derived from content: `active`, `resolved`, `waiting`, `escalated` |
| categories | jsonb | Auto-categorized topics |
| stats | jsonb | Summary statistics (message counts, etc.) |
| summary_type | string | Summary type: `conversation` (full summary) or `digest` (recent messages only) |
| prompt_id | string | Jarvis prompt used for generation |
| message_count | integer | Number of messages processed |
| last_processed_message_id | string | ID of the last message included in summary |
| tokens_used | integer | Tokens consumed for generation |
| validation_status | string | Output validation: `valid`, `repaired`, `raw_fallback` |
| retry_count | integer | Number of LLM retries for valid output |
| created_at | datetime | Summary creation timestamp |
| updated_at | datetime | Last update timestamp |

### Task

> **Relationship:** Conversations have a `tasks` (has_many) relationship. Include via `?include=tasks`. Tasks are persistent action items extracted from AI summaries. A 7-day deduplication window prevents duplicate tasks from being created.

| Field | Type | Description |
|-------|------|-------------|
| unique_id | uuid | Unique identifier for the task |
| description | string | Task description |
| priority | string | Task priority: `normal`, `high`, `urgent` |
| status | string | Task status: `pending`, `completed`, `dismissed` |
| conversation_context_id | uuid | Parent conversation ID |
| created_at | datetime | Task creation timestamp |
| updated_at | datetime | Last update timestamp |

> **Including relationships in conversation response:**
> ```
> GET /conversations/:id?include=summary,tasks
> ```

---

## WebSocket Channels

### UserInboxChannel

Subscribe to real-time notifications when a user receives new conversations.

**Subscribe:**
```json
{ "channel": "UserInboxChannel", "user_id": "<user_unique_id>" }
```

**Events:**
| Event | Description |
|-------|-------------|
| `new_conversation` | A new direct conversation was created involving this user |
| `new_group_conversation` | A new group conversation was created involving this user |

---

## Breaking Changes

> **Read status is now per-user via read horizon.** The conversation-level `unread_count` is now computed per-user using individual `MessageReadReceipt` records and a `last_read_at` read horizon on `context_users`. The old behavior where message `status` changed to `'read'` globally is no longer in effect. See the **23blocks-conversations-read-receipts-api** skill for details.

> **Digest endpoint changed (v1.4).** `POST /conversations/digest` is now `GET /users/:id/conversations/summary`. Response structure changed: `attributes.digest` contains `{ summary, categories, action_items, stats }`, `attributes.content` has raw LLM output, `attributes.meta` includes `validation_status` and `retry_count`. The `conversations_found` boolean is now always present in digest responses — use it to distinguish "query found no conversations" (`false`) from "Jarvis failed to summarize" (`true` but empty summary).

## New Features

### First Response Tracking

The `first_response_tracking` field on contexts tracks when the first response was sent in a conversation, useful for SLA and response time metrics.

### Auto-Read on Show

When a conversation is retrieved via `GET /conversations/:unique_id`, all messages are automatically marked as read for the requesting user via `MessageReadService.mark_conversation_as_read`.

### Unread Counter Accuracy (v1.3)

`group_users.unread_count` is the single source of truth — incremented on message create, zeroed on conversation view. Viewing a conversation now correctly resets the counter.

### AI Conversation Summaries (v1.3)

Generate AI-powered summaries via Jarvis integration. Supports incremental processing (only new messages since last summary), custom prompts via `prompt_id`, and batch digest for inbox-level overviews. Rate limited to 1 Jarvis call per 60s per conversation per user.

> **Jarvis Passthrough Auth (v1.4).** The Conversations API no longer looks up per-tenant CompanyKeys for Jarvis. Consumer `Authorization` (Bearer JWT) and `X-API-Key` headers are forwarded directly to Jarvis as-is. The same credentials that authenticate with the Conversations API now authenticate with Jarvis — no separate Jarvis key is needed.

### Summary Relationships (v1.5)

Conversations now expose a `summary` relationship (has_one) that can be included via `?include=summary`. Summaries are user-scoped, so each user gets their own perspective. The summary includes `key_points` (key discussion points) in addition to the existing `action_items`.

### Task Management (v1.5)

Conversations now support persistent task management. Tasks are action items extracted from AI summaries or created manually. Each task has a `priority` (`normal`, `high`, `urgent`) and `status` (`pending`, `completed`, `dismissed`). A 7-day deduplication window prevents duplicate tasks. Tasks can be included in conversation responses via `?include=tasks`, or queried per-conversation or per-user for a full task digest.

Task lifecycle transitions use `PUT /tasks/:uid?action_type=complete|dismiss|reopen`. Attribute updates (description, priority) use `PUT /tasks/:uid` with a request body. All updates use PUT (not PATCH) per platform convention.

User task digest (`GET /users/:uid/tasks`) supports filtering by `status`, `context_unique_id`, `reference`, `source`, `source_type`, `source_id`.

### Digest Persistence (v1.5)

`GET /users/:uid/conversations/summary` now returns the last completed digest when the user has zero unread messages, preventing the digest from "disappearing" after the user catches up.

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
