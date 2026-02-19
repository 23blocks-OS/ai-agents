---
name: jarvis-prompt-versions-api
description: Manage 23blocks Jarvis prompt versions via REST API. Use when listing prompt versions, publishing versions, executing prompts with LLM providers, or streaming prompt execution results.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Prompt Versions API

Complete API reference for 23blocks Jarvis prompt versioning with publishing, execution, and streaming.

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
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid/versions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /prompts/:id/versions - List Versions

Lists all versions for a prompt.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/versions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "variables": {
      "tone": "professional",
      "topic": "quarterly results"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `variables` | object | No | Template variable values |
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
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "variables": {
      "tone": "professional",
      "topic": "quarterly results"
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
