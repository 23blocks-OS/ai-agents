---
name: 23blocks-jarvis-prompts-api
description: Manage 23blocks Jarvis prompts via REST API. Use when creating prompts, rendering prompts with variables, or managing prompt social interactions (likes, follows, saves).
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.2"
---

# Prompts API

Complete API reference for 23blocks Jarvis prompt management with rendering and social interactions.

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

## Breaking Changes & Bug Fixes

> **Breaking Change (May 2026):** POST and PUT responses now return `PromptVersion` objects instead of `Prompt` objects. The Location header and response body now both reference the created PromptVersion.

> **Bug Fix (May 2026):** Partial PUT updates now preserve unpassed fields (previously wiped persona, model, temperature when not included in request).

---

## Endpoints

### GET /prompts - List Prompts

Lists all prompts with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts?page=1&records=20" \
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
      "id": "prompt-uuid-123",
      "type": "prompt",
      "attributes": {
        "unique_id": "prompt-uuid-123",
        "name": "Email Generator",
        "description": "Generates professional emails",
        "content": "Write a {{tone}} email about {{topic}} to {{recipient}}.",
        "status": "published",
        "likes_count": 15,
        "saves_count": 8,
        "versions_count": 3,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### GET /prompts/:id - Get Prompt

Retrieves a single prompt by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "prompt-uuid-123",
    "type": "prompt",
    "attributes": {
      "unique_id": "prompt-uuid-123",
      "name": "Email Generator",
      "description": "Generates professional emails",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}.",
      "status": "published",
      "likes_count": 15,
      "saves_count": 8,
      "versions_count": 3,
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "versions": {
        "data": [
          { "id": "version-uuid-1", "type": "prompt_version" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Prompt not found

---

### POST /prompts - Create Prompt

Creates a new prompt.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": {
      "name": "Email Generator",
      "description": "Generates professional emails based on tone and topic",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}.",
      "provider": "anthropic",
      "model": "claude-sonnet-4-5-20241022"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Prompt name |
| `description` | string | No | Prompt description |
| `content` | string | Yes | Prompt template with {{variables}} |
| `provider` | string | No | LLM provider: `openai` (default), `anthropic`, `google`, `mistral`, `perplexity`, `openai_compatible`, `custom` |
| `model` | string | No | Model identifier for the provider (e.g., `gpt-4`, `claude-sonnet-4-5-20241022`, `mistral-small-latest`) |

**Response 201:**

> **Breaking Change (May 2026):** This endpoint now returns a `PromptVersion` object instead of a `Prompt` object.

```json
{
  "data": {
    "id": "version-uuid-001",
    "type": "prompt_version",
    "attributes": {
      "unique_id": "version-uuid-001",
      "name": "Email Generator",
      "description": "Generates professional emails based on tone and topic",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}.",
      "status": "draft",
      "version": 1,
      "revision": 0,
      "prompt_unique_id": "prompt-uuid-123",
      "user_unique_id": "user-uuid-456",
      "user_name": "John Doe",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /prompts/:id - Update Prompt

Updates an existing prompt. Only the fields included in the request body are updated; omitted fields are preserved (e.g., persona, model, temperature are no longer wiped when not passed).

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": {
      "name": "Advanced Email Generator",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}. Include a {{call_to_action}}."
    }
  }'
```

**Response 200:**

> **Breaking Change (May 2026):** This endpoint now returns a `PromptVersion` object instead of a `Prompt` object.

```json
{
  "data": {
    "id": "version-uuid-002",
    "type": "prompt_version",
    "attributes": {
      "unique_id": "version-uuid-002",
      "name": "Advanced Email Generator",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}. Include a {{call_to_action}}.",
      "version": 2,
      "revision": 0,
      "prompt_unique_id": "prompt-uuid-123",
      "user_unique_id": "user-uuid-456",
      "user_name": "John Doe",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /prompts/:id - Delete Prompt

Deletes a prompt.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Prompt not found

> **Required Scopes:** POST, PUT, and DELETE `/prompts` endpoints require the `prompts:write` scope.

---

### POST /prompts/:id/render - Render Prompt

Renders a prompt template by substituting variables.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/render" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "variables": {
      "tone": "professional",
      "topic": "quarterly results",
      "recipient": "the board of directors"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `variables` | object | Yes | Key-value pairs for template variables |

**Response 200:**
```json
{
  "data": {
    "rendered_content": "Write a professional email about quarterly results to the board of directors.",
    "variables_used": ["tone", "topic", "recipient"],
    "missing_variables": []
  }
}
```

---

## Social Actions

### PUT /prompts/:id/like - Like Prompt

Adds a like to the prompt.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/like" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt liked successfully"
}
```

---

### DELETE /prompts/:id/dislike - Remove Like

Removes like from the prompt.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/dislike" \
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

### PUT /prompts/:id/follow - Follow Prompt

Follows a prompt to receive updates.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/follow" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt followed successfully"
}
```

---

### DELETE /prompts/:id/unfollow - Unfollow Prompt

Unfollows a prompt.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/unfollow" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt unfollowed successfully"
}
```

---

### PUT /prompts/:id/save - Save Prompt

Saves/bookmarks a prompt.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/save" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt saved successfully"
}
```

---

### DELETE /prompts/:id/unsave - Unsave Prompt

Removes prompt from saved list.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/unsave" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt unsaved successfully"
}
```

---

## Data Models

### Prompt
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Prompt name |
| `description` | string | Prompt description |
| `content` | string | Template with {{variables}} |
| `provider` | string | LLM provider (`openai`, `anthropic`, `google`, `mistral`, `perplexity`, `openai_compatible`, `custom`) |
| `model` | string | Model identifier for the provider |
| `status` | enum | draft, published |
| `likes_count` | integer | Number of likes |
| `saves_count` | integer | Number of saves |
| `versions_count` | integer | Number of versions |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### PromptVersion (returned by POST/PUT since May 2026)
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier of the version |
| `name` | string | Prompt name |
| `description` | string | Prompt description |
| `content` | string | Template with {{variables}} |
| `status` | enum | draft, published |
| `version` | integer | Major version number |
| `revision` | integer | Revision within the version |
| `prompt_unique_id` | uuid | Parent prompt unique ID |
| `user_unique_id` | uuid | Creator user unique ID |
| `user_name` | string | Creator display name |
| `is_published` | boolean | Whether currently published |
| `variables` | array | List of template variables |
| `executions_count` | integer | Number of executions |
| `published_at` | timestamp | Publication time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Prompt Not Found",
    "detail": "The requested prompt could not be found."
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
// PromptsService — client.jarvis.prompts
list(params?: ListPromptsParams): Promise<PageResult<Prompt>>;
get(uniqueId: string): Promise<Prompt>;
create(data: CreatePromptRequest): Promise<PromptVersion>;   // Breaking change: returns PromptVersion since May 2026
update(uniqueId: string, data: UpdatePromptRequest): Promise<PromptVersion>;   // Breaking change: returns PromptVersion since May 2026
delete(uniqueId: string): Promise<void>;
execute(uniqueId: string, data: ExecutePromptRequest): Promise<ExecutePromptResponse>;
render(uniqueId: string, data: RenderPromptRequest): Promise<RenderPromptResponse>;
```

### TypeScript Types

```typescript
import type {
  Prompt,
  PromptVersion,
  CreatePromptRequest,
  UpdatePromptRequest,
  ListPromptsParams,
  ExecutePromptRequest,
  ExecutePromptResponse,
  RenderPromptRequest,
  RenderPromptResponse,
  RenderPromptMeta,
  PlaceholderValue,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();

  // Example: render a prompt with variables
  const result = await client.jarvis.prompts.render('prompt-uuid', {
    placeholders: { tone: 'professional', topic: 'quarterly results' },
  });
}
```
