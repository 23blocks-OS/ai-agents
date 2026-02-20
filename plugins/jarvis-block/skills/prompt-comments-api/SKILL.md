---
name: jarvis-prompt-comments-api
description: Manage 23blocks Jarvis prompt comments via REST API. Use when adding comments to prompts, managing execution comments, or handling comment social interactions (likes).
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Prompt Comments API

Complete API reference for 23blocks Jarvis prompt and execution comment management with social interactions.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://jarvis.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Jarvis API base URL | `https://jarvis.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Prompt Comments

### GET /prompts/:id/comments - List Prompt Comments

Lists all comments on a prompt.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "comment-uuid-456",
      "type": "comment",
      "attributes": {
        "unique_id": "comment-uuid-456",
        "body": "Great prompt! Works well for sales emails.",
        "author_id": "user-uuid-789",
        "author_name": "Jane Smith",
        "likes_count": 5,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /prompts/:id/comments/:comment_id - Get Comment

Retrieves a single prompt comment.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments/comment-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "comment-uuid-456",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-456",
      "body": "Great prompt! Works well for sales emails.",
      "author_id": "user-uuid-789",
      "author_name": "Jane Smith",
      "likes_count": 5,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Comment not found

---

### POST /prompts/:id/comments - Create Comment

Adds a comment to a prompt.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "Great prompt! Works well for sales emails."
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `body` | string | Yes | Comment text |

**Response 201:**
```json
{
  "data": {
    "id": "comment-uuid-456",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-456",
      "body": "Great prompt! Works well for sales emails.",
      "likes_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /prompts/:id/comments/:comment_id - Update Comment

Updates an existing comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments/comment-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "Updated: Great prompt! Especially effective for formal sales emails."
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "comment-uuid-456",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-456",
      "body": "Updated: Great prompt! Especially effective for formal sales emails.",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /prompts/:id/comments/:comment_id - Delete Comment

Deletes a comment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments/comment-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /prompts/:id/comments/:comment_id/like - Like Comment

Likes a prompt comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments/comment-uuid-456/like" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Comment liked successfully"
}
```

---

### DELETE /prompts/:id/comments/:comment_id/dislike - Remove Like

Removes like from a prompt comment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments/comment-uuid-456/dislike" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Like removed successfully"
}
```

---

## Execution Comments

### GET /prompts/:id/executions/:exec_id/comments - List Execution Comments

Lists all comments on a prompt execution.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "comment-uuid-101",
      "type": "comment",
      "attributes": {
        "unique_id": "comment-uuid-101",
        "body": "This execution produced excellent results.",
        "author_id": "user-uuid-789",
        "author_name": "Jane Smith",
        "likes_count": 3,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /prompts/:id/executions/:exec_id/comments - Create Execution Comment

Adds a comment to a prompt execution.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "This execution produced excellent results."
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "comment-uuid-101",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-101",
      "body": "This execution produced excellent results.",
      "likes_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /prompts/:id/executions/:exec_id/comments/:comment_id - Update Execution Comment

Updates an execution comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments/comment-uuid-101" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "Updated: Excellent results, especially the tone."
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "comment-uuid-101",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-101",
      "body": "Updated: Excellent results, especially the tone.",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /prompts/:id/executions/:exec_id/comments/:comment_id - Delete Execution Comment

Deletes an execution comment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments/comment-uuid-101" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /prompts/:id/executions/:exec_id/comments/:comment_id/like - Like Execution Comment

Likes an execution comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments/comment-uuid-101/like" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Comment liked successfully"
}
```

---

### DELETE /prompts/:id/executions/:exec_id/comments/:comment_id/dislike - Remove Like

Removes like from an execution comment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments/comment-uuid-101/dislike" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Like removed successfully"
}
```

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
