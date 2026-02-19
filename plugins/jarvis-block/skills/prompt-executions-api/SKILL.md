---
name: jarvis-prompt-executions-api
description: Manage 23blocks Jarvis prompt executions via REST API. Use when viewing execution history, retrieving execution details, or managing execution social interactions (likes, saves).
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Prompt Executions API

Complete API reference for 23blocks Jarvis prompt execution history with social interactions.

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
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid/executions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
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
