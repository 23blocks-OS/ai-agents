---
name: 23blocks-jarvis-prompt-executions-api
description: "Jarvis prompt execution history: list, details, like, save. Use when reviewing past prompt runs."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Prompt Executions API

Complete API reference for 23blocks Jarvis prompt execution history with social interactions.

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

### GET /prompts/:id/executions - List Executions

Lists all executions for a prompt with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Execution Not Found","detail":"The requested execution could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
