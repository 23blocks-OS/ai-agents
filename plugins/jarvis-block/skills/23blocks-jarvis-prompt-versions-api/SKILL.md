---
name: 23blocks-jarvis-prompt-versions-api
description: Manage 23blocks Jarvis prompt versions via REST API. Use when listing prompt versions, publishing versions, executing prompts with LLM providers, or streaming prompt execution results.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Prompt Versions API

Complete API reference for 23blocks Jarvis prompt versioning with publishing, execution, and streaming.

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

### GET /prompts/:id/versions - List Versions

Lists all versions for a prompt.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/versions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "version-uuid-1",
      "type": "prompt_version",
      "attributes": {
        "unique_id": "version-uuid-1",
        "version_number": 1,
        "content": "Write a {{tone}} email about {{topic}}.",
        "status": "published",
        "is_published": true,
        "executions_count": 45,
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "version-uuid-2",
      "type": "prompt_version",
      "attributes": {
        "unique_id": "version-uuid-2",
        "version_number": 2,
        "content": "Write a {{tone}} email about {{topic}} to {{recipient}}.",
        "status": "draft",
        "is_published": false,
        "executions_count": 3,
        "created_at": "2025-01-12T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /prompts/:id/versions/:version_id - Get Version

Retrieves a single prompt version.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/versions/version-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "version-uuid-1",
    "type": "prompt_version",
    "attributes": {
      "unique_id": "version-uuid-1",
      "version_number": 1,
      "content": "Write a {{tone}} email about {{topic}}.",
      "status": "published",
      "is_published": true,
      "executions_count": 45,
      "variables": ["tone", "topic"],
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Version not found

---

### POST /prompts/:id/versions/:version_id/publish - Publish Version

Publishes a prompt version for production use.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/versions/version-uuid-2/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "version-uuid-2",
    "type": "prompt_version",
    "attributes": {
      "unique_id": "version-uuid-2",
      "version_number": 2,
      "status": "published",
      "is_published": true,
      "published_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### POST /prompts/:id/versions/:version_id/execute - Execute Version

Executes a prompt version with an LLM provider.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/versions/version-uuid-1/execute" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": {
      "content": "Write a professional email about quarterly results",
      "additional_data": "{\"tone\":\"professional\",\"topic\":\"quarterly results\"}"
    }
  }'
```

**Request Parameters (wrapped in `"prompt"` key):**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `content` | string | Yes | The prompt content / user message to execute |
| `additional_data` | string | No | JSON-stringified additional variables/context |
| `model` | string | No | Override LLM model |
| `temperature` | float | No | Override temperature (0-1) |

**Response 200:**
```json
{
  "data": {
    "id": "exec-uuid-789",
    "type": "prompt_execution",
    "attributes": {
      "unique_id": "exec-uuid-789",
      "version_id": "version-uuid-1",
      "rendered_prompt": "Write a professional email about quarterly results.",
      "output": "Subject: Q4 Results Summary\n\nDear Team,\n\nI am pleased to share our quarterly results...",
      "model": "gpt-4",
      "tokens_used": 320,
      "duration_ms": 2100,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /prompts/:id/versions/:version_id/execute/stream - Stream Execution

Executes a prompt version and streams the response in real-time.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/versions/version-uuid-1/execute/stream" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": {
      "content": "Write a professional email about quarterly results",
      "additional_data": "{\"tone\":\"professional\",\"topic\":\"quarterly results\"}"
    }
  }'
```

**Response 200 (Server-Sent Events):**
```
data: {"type":"token","content":"Subject:"}
data: {"type":"token","content":" Q4"}
data: {"type":"token","content":" Results"}
data: {"type":"token","content":" Summary"}
data: {"type":"done","execution_id":"exec-uuid-789","tokens_used":320}
```

---

## Data Models

### PromptVersion
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `version_number` | integer | Sequential version number |
| `content` | string | Prompt template content |
| `status` | enum | draft, published |
| `is_published` | boolean | Whether currently published |
| `variables` | array | List of template variables |
| `executions_count` | integer | Number of executions |
| `published_at` | timestamp | Publication time |
| `created_at` | timestamp | Creation time |

### PromptExecution
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `version_id` | uuid | Source version ID |
| `rendered_prompt` | string | Rendered prompt text |
| `output` | string | LLM response |
| `model` | string | Model used |
| `tokens_used` | integer | Tokens consumed |
| `duration_ms` | integer | Execution duration |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Version Not Found",
    "detail": "The requested prompt version could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

**Note:** The Prompt Versions API does not have a dedicated SDK service yet. Use raw API calls as documented above, or use the generic HTTP client from the SDK. Prompt execution is available via `client.jarvis.prompts.execute()`.

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

### TypeScript Types

```typescript
import type {
  ExecutePromptVersionRequest,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();

  // Prompt versions are managed via REST API calls.
  // For prompt execution, use client.jarvis.prompts.execute().
  const result = await client.jarvis.prompts.execute('prompt-uuid', {
    agentUniqueId: 'agent-uuid',
    variables: { tone: 'professional' },
  });
}
```
