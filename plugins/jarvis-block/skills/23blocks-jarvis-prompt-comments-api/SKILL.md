---
name: 23blocks-jarvis-prompt-comments-api
description: "Jarvis comments on prompts and executions, with likes. Use for prompt review discussions."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Prompt Comments API

Complete API reference for 23blocks Jarvis prompt and execution comment management with social interactions.

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Comment Not Found","detail":"The requested comment could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
