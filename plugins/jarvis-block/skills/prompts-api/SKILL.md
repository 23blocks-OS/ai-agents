---
name: jarvis-prompts-api
description: Manage 23blocks Jarvis prompts via REST API. Use when creating prompts, rendering prompts with variables, or managing prompt social interactions (likes, follows, saves).
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Prompts API

Complete API reference for 23blocks Jarvis prompt management with rendering and social interactions.

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
curl -X GET "$BLOCKS_API_URL/prompts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /prompts - List Prompts

Lists all prompts with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts?page=1&records=20" \
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
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": {
      "name": "Email Generator",
      "description": "Generates professional emails based on tone and topic",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}."
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Prompt name |
| `description` | string | No | Prompt description |
| `content` | string | Yes | Prompt template with {{variables}} |

**Response 201:**
```json
{
  "data": {
    "id": "prompt-uuid-123",
    "type": "prompt",
    "attributes": {
      "unique_id": "prompt-uuid-123",
      "name": "Email Generator",
      "description": "Generates professional emails based on tone and topic",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}.",
      "status": "draft",
      "versions_count": 1,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /prompts/:id - Update Prompt

Updates an existing prompt.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": {
      "name": "Advanced Email Generator",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}. Include a {{call_to_action}}."
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "prompt-uuid-123",
    "type": "prompt",
    "attributes": {
      "unique_id": "prompt-uuid-123",
      "name": "Advanced Email Generator",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}. Include a {{call_to_action}}.",
      "versions_count": 2,
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
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Prompt not found

---

### POST /prompts/:id/render - Render Prompt

Renders a prompt template by substituting variables.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/render" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
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
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
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
| `status` | enum | draft, published |
| `likes_count` | integer | Number of likes |
| `saves_count` | integer | Number of saves |
| `versions_count` | integer | Number of versions |
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
create(data: CreatePromptRequest): Promise<Prompt>;
update(uniqueId: string, data: UpdatePromptRequest): Promise<Prompt>;
delete(uniqueId: string): Promise<void>;
execute(uniqueId: string, data: ExecutePromptRequest): Promise<ExecutePromptResponse>;
render(uniqueId: string, data: RenderPromptRequest): Promise<RenderPromptResponse>;
```

### TypeScript Types

```typescript
import type {
  Prompt,
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
