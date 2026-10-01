---
name: 23blocks-jarvis-prompt-versions-api
description: "Jarvis prompt versions: list, publish, execute against an LLM, stream output. Use to run a prompt."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.1"
---

# Prompt Versions API

Complete API reference for 23blocks Jarvis prompt versioning with publishing, execution, and streaming.

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

## Behaviour notes

`POST /prompts` and `PUT /prompts/:id` return a `PromptVersion`, not a `Prompt` (see the `23blocks-jarvis-prompts-api` skill). A `PromptVersion` carries `version` (major version number), `revision` (revision within the version), `prompt_unique_id`, `user_unique_id` and `user_name` (creator).

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
| `version` | integer | Major version number |
| `revision` | integer | Revision within the version |
| `prompt_unique_id` | uuid | Parent prompt unique ID |
| `user_unique_id` | uuid | Creator user unique ID |
| `user_name` | string | Creator display name |
| `name` | string | Prompt name |
| `description` | string | Prompt description |
| `content` | string | Prompt template content |
| `status` | enum | draft, published |
| `is_published` | boolean | Whether currently published |
| `variables` | array | List of template variables |
| `executions_count` | integer | Number of executions |
| `published_at` | timestamp | Publication time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Version Not Found","detail":"The requested prompt version could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
