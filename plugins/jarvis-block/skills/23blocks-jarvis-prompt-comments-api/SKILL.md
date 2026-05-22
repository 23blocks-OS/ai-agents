---
name: 23blocks-jarvis-prompt-comments-api
description: Manage 23blocks Jarvis prompt comments via REST API. Use when adding comments to prompts, managing execution comments, or handling comment social interactions (likes).
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Prompt Comments API

Complete API reference for 23blocks Jarvis prompt and execution comment management with social interactions.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Jarvis API base URL | `https://jarvis.api.us.23blocks.com` |
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

---

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
| GET | `/prompts/:id/comments` | List prompt comments |
| GET | `/prompts/:id/comments/:comment_id` | Get a comment |
| POST | `/prompts/:id/comments` | Create a comment |
| PUT | `/prompts/:id/comments/:comment_id` | Update a comment |
| DELETE | `/prompts/:id/comments/:comment_id` | Delete a comment |
| PUT | `/prompts/:id/comments/:comment_id/like` | Like a comment |
| DELETE | `/prompts/:id/comments/:comment_id/dislike` | Remove like from comment |
| GET | `/prompts/:id/executions/:exec_id/comments` | List execution comments |
| POST | `/prompts/:id/executions/:exec_id/comments` | Create execution comment |
| PUT | `/prompts/:id/executions/:exec_id/comments/:comment_id` | Update execution comment |
| DELETE | `/prompts/:id/executions/:exec_id/comments/:comment_id` | Delete execution comment |
| PUT | `/prompts/:id/executions/:exec_id/comments/:comment_id/like` | Like execution comment |
| DELETE | `/prompts/:id/executions/:exec_id/comments/:comment_id/dislike` | Remove like from execution comment |

---

## Data Models

### Comment
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `body` | string | Comment text |
| `author_id` | uuid | Author user ID |
| `author_name` | string | Author display name |
| `likes_count` | integer | Number of likes |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Comment Not Found",
    "detail": "The requested comment could not be found."
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
// PromptCommentsService — client.jarvis.promptComments
list(promptUniqueId: string, params?: ListPromptCommentsParams): Promise<PageResult<PromptComment>>;
get(promptUniqueId: string, uniqueId: string): Promise<PromptComment>;
create(promptUniqueId: string, data: CreatePromptCommentRequest): Promise<PromptComment>;
update(promptUniqueId: string, uniqueId: string, data: UpdatePromptCommentRequest): Promise<PromptComment>;
delete(promptUniqueId: string, uniqueId: string): Promise<void>;
like(promptUniqueId: string, uniqueId: string): Promise<void>;
dislike(promptUniqueId: string, uniqueId: string): Promise<void>;
reply(promptUniqueId: string, uniqueId: string, data: ReplyToCommentRequest): Promise<PromptComment>;
follow(promptUniqueId: string, uniqueId: string): Promise<void>;
unfollow(promptUniqueId: string, uniqueId: string): Promise<void>;
save(promptUniqueId: string, uniqueId: string): Promise<void>;
unsave(promptUniqueId: string, uniqueId: string): Promise<void>;

// ExecutionCommentsService — client.jarvis.executionComments
list(promptUniqueId: string, executionUniqueId: string, params?: ListExecutionCommentsParams): Promise<PageResult<ExecutionComment>>;
get(promptUniqueId: string, executionUniqueId: string, uniqueId: string): Promise<ExecutionComment>;
create(promptUniqueId: string, executionUniqueId: string, data: CreateExecutionCommentRequest): Promise<ExecutionComment>;
update(promptUniqueId: string, executionUniqueId: string, uniqueId: string, data: UpdateExecutionCommentRequest): Promise<ExecutionComment>;
delete(promptUniqueId: string, executionUniqueId: string, uniqueId: string): Promise<void>;
like(promptUniqueId: string, executionUniqueId: string, uniqueId: string): Promise<void>;
dislike(promptUniqueId: string, executionUniqueId: string, uniqueId: string): Promise<void>;
reply(promptUniqueId: string, executionUniqueId: string, uniqueId: string, data: ReplyToCommentRequest): Promise<ExecutionComment>;
follow(promptUniqueId: string, executionUniqueId: string, uniqueId: string): Promise<void>;
unfollow(promptUniqueId: string, executionUniqueId: string, uniqueId: string): Promise<void>;
save(promptUniqueId: string, executionUniqueId: string, uniqueId: string): Promise<void>;
unsave(promptUniqueId: string, executionUniqueId: string, uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  PromptComment,
  CreatePromptCommentRequest,
  UpdatePromptCommentRequest,
  ListPromptCommentsParams,
  ReplyToCommentRequest,
  ExecutionComment,
  CreateExecutionCommentRequest,
  UpdateExecutionCommentRequest,
  ListExecutionCommentsParams,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();

  // Example: list comments for a prompt
  const comments = await client.jarvis.promptComments.list('prompt-uuid');
}
```
