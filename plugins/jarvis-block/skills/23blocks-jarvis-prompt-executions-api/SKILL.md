---
name: 23blocks-jarvis-prompt-executions-api
description: Manage 23blocks Jarvis prompt executions via REST API. Use when viewing execution history, retrieving execution details, or managing execution social interactions (likes, saves).
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Prompt Executions API

Complete API reference for 23blocks Jarvis prompt execution history with social interactions.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Jarvis API base URL | `https://jarvis.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

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

## Endpoints

### GET /prompts/:id/executions - List Executions

Lists all executions for a prompt with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "exec-uuid-789",
      "type": "prompt_execution",
      "attributes": {
        "unique_id": "exec-uuid-789",
        "version_id": "version-uuid-1",
        "version_number": 1,
        "rendered_prompt": "Write a professional email about quarterly results.",
        "output": "Subject: Q4 Results Summary...",
        "model": "gpt-4",
        "tokens_used": 320,
        "duration_ms": 2100,
        "likes_count": 5,
        "saves_count": 2,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 45
  }
}
```

---

### GET /prompts/:id/executions/:exec_id - Get Execution

Retrieves a single execution.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "exec-uuid-789",
    "type": "prompt_execution",
    "attributes": {
      "unique_id": "exec-uuid-789",
      "version_id": "version-uuid-1",
      "version_number": 1,
      "rendered_prompt": "Write a professional email about quarterly results.",
      "output": "Subject: Q4 Results Summary\n\nDear Team,\n\nI am pleased to share our quarterly results...",
      "variables": {"tone": "professional", "topic": "quarterly results"},
      "model": "gpt-4",
      "temperature": 0.7,
      "tokens_used": 320,
      "duration_ms": 2100,
      "likes_count": 5,
      "saves_count": 2,
      "comments_count": 3,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Execution not found

---

## Social Actions

### PUT /prompts/:id/executions/:exec_id/like - Like Execution

Likes a prompt execution.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/like" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Execution liked successfully"
}
```

---

### DELETE /prompts/:id/executions/:exec_id/dislike - Remove Like

Removes like from an execution.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/dislike" \
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

### PUT /prompts/:id/executions/:exec_id/save - Save Execution

Saves/bookmarks an execution.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/save" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Execution saved successfully"
}
```

---

### DELETE /prompts/:id/executions/:exec_id/unsave - Unsave Execution

Removes execution from saved list.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/unsave" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Execution unsaved successfully"
}
```

---

## Data Models

### PromptExecution
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `version_id` | uuid | Source version ID |
| `version_number` | integer | Version number |
| `rendered_prompt` | string | Rendered prompt text |
| `output` | string | LLM response |
| `variables` | object | Variables used |
| `model` | string | LLM model used |
| `temperature` | float | Temperature setting |
| `tokens_used` | integer | Tokens consumed |
| `duration_ms` | integer | Execution duration |
| `likes_count` | integer | Number of likes |
| `saves_count` | integer | Number of saves |
| `comments_count` | integer | Number of comments |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Execution Not Found",
    "detail": "The requested execution could not be found."
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
// ExecutionsService — client.jarvis.executions
list(params?: ListExecutionsParams): Promise<PageResult<Execution>>;
get(uniqueId: string): Promise<Execution>;
listByAgent(agentUniqueId: string, params?: ListExecutionsParams): Promise<PageResult<Execution>>;
listByPrompt(promptUniqueId: string, params?: ListExecutionsParams): Promise<PageResult<Execution>>;
cancel(uniqueId: string): Promise<Execution>;
```

### TypeScript Types

```typescript
import type {
  Execution,
  ListExecutionsParams,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();

  // Example: list executions for a prompt
  const executions = await client.jarvis.executions.listByPrompt('prompt-uuid');
}
```
