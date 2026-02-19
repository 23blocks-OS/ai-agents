---
name: jarvis-agent-threads-api
description: Manage 23blocks Jarvis agent threads, messages, and runs via REST API. Use when creating agent conversation threads, sending messages, streaming responses, or managing agent execution runs.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Agent Threads API

Complete API reference for 23blocks Jarvis agent runtime — threads, messages, streaming, runs, and executions.

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
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid/threads" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Agent Context

### GET /agents/:id/context - Get Agent Context

Retrieves the aggregated context for an agent (system prompt, entity contexts, prompts).

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/context" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "agent-uuid-123",
    "type": "agent_context",
    "attributes": {
      "system_prompt": "You are a helpful customer support agent.",
      "entity_contexts": [
        {
          "entity_id": "entity-uuid-1",
          "name": "Product KB",
          "content": "Product documentation..."
        }
      ],
      "prompts": [
        {
          "prompt_id": "prompt-uuid-1",
          "name": "Greeting Template",
          "content": "Hello {{name}}, how can I help you today?"
        }
      ]
    }
  }
}
```

---

## Threads

### GET /agents/:id/threads - List Threads

Lists all threads for an agent.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/threads?page=1&records=20" \
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
      "id": "thread-uuid-456",
      "type": "thread",
      "attributes": {
        "unique_id": "thread-uuid-456",
        "title": "Support Session #1",
        "messages_count": 12,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T11:45:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 68
  }
}
```

---

### GET /agents/:id/threads/:thread_id - Get Thread

Retrieves a single thread.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/threads/thread-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "thread-uuid-456",
    "type": "thread",
    "attributes": {
      "unique_id": "thread-uuid-456",
      "title": "Support Session #1",
      "messages_count": 12,
      "runs_count": 6,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Thread not found

---

### POST /agents/:id/threads - Create Thread

Creates a new conversation thread for an agent.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/threads" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "thread": {
      "title": "Support Session #1"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | No | Thread title |

**Response 201:**
```json
{
  "data": {
    "id": "thread-uuid-456",
    "type": "thread",
    "attributes": {
      "unique_id": "thread-uuid-456",
      "title": "Support Session #1",
      "messages_count": 0,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /agents/:id/threads/:thread_id - Delete Thread

Deletes a thread and its messages.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/agents/agent-uuid-123/threads/thread-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Thread Messages

### GET /agents/:id/threads/:thread_id/messages - List Messages

Lists messages in a thread.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/threads/thread-uuid-456/messages?page=1&records=50" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "msg-uuid-101",
      "type": "message",
      "attributes": {
        "unique_id": "msg-uuid-101",
        "role": "user",
        "content": "How do I reset my password?",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "msg-uuid-102",
      "type": "message",
      "attributes": {
        "unique_id": "msg-uuid-102",
        "role": "assistant",
        "content": "To reset your password, go to Settings > Security > Reset Password.",
        "created_at": "2025-01-10T10:30:05Z"
      }
    }
  ]
}
```

---

### POST /agents/:id/threads/:thread_id/messages - Send Message

Sends a message in a thread.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/threads/thread-uuid-456/messages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "role": "user",
      "content": "How do I reset my password?"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `role` | string | Yes | Message role (user, assistant, system) |
| `content` | string | Yes | Message content |

**Response 201:**
```json
{
  "data": {
    "id": "msg-uuid-101",
    "type": "message",
    "attributes": {
      "unique_id": "msg-uuid-101",
      "role": "user",
      "content": "How do I reset my password?",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /agents/:id/threads/:thread_id/messages/stream - Stream Message

Sends a message and streams the AI response in real-time.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/threads/thread-uuid-456/messages/stream" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "content": "Explain the refund policy in detail."
    }
  }'
```

**Response 200 (Server-Sent Events):**
```
data: {"type":"token","content":"Our"}
data: {"type":"token","content":" refund"}
data: {"type":"token","content":" policy"}
data: {"type":"token","content":" allows"}
data: {"type":"done","message_id":"msg-uuid-103"}
```

---

## Runs

### POST /agents/:id/threads/:thread_id/runs - Create Run

Creates a new agent execution run in a thread.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/threads/thread-uuid-456/runs" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "run": {
      "message": "How do I reset my password?"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `message` | string | Yes | Input message for the run |

**Response 201:**
```json
{
  "data": {
    "id": "run-uuid-789",
    "type": "run",
    "attributes": {
      "unique_id": "run-uuid-789",
      "status": "queued",
      "input_message": "How do I reset my password?",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### GET /agents/:id/threads/:thread_id/runs - List Runs

Lists all runs in a thread.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/threads/thread-uuid-456/runs" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "run-uuid-789",
      "type": "run",
      "attributes": {
        "unique_id": "run-uuid-789",
        "status": "completed",
        "input_message": "How do I reset my password?",
        "output_message": "To reset your password...",
        "tokens_used": 245,
        "duration_ms": 1250,
        "created_at": "2025-01-10T10:30:00Z",
        "completed_at": "2025-01-10T10:30:02Z"
      }
    }
  ]
}
```

---

### GET /agents/:id/threads/:thread_id/runs/:run_id - Get Run

Retrieves a single run.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/threads/thread-uuid-456/runs/run-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "run-uuid-789",
    "type": "run",
    "attributes": {
      "unique_id": "run-uuid-789",
      "status": "completed",
      "input_message": "How do I reset my password?",
      "output_message": "To reset your password, go to Settings > Security > Reset Password.",
      "tokens_used": 245,
      "duration_ms": 1250,
      "model": "gpt-4",
      "created_at": "2025-01-10T10:30:00Z",
      "completed_at": "2025-01-10T10:30:02Z"
    }
  }
}
```

---

### GET /agents/:id/threads/:thread_id/runs/:run_id/executions - List Run Executions

Lists execution steps within a run.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/threads/thread-uuid-456/runs/run-uuid-789/executions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "exec-uuid-101",
      "type": "execution",
      "attributes": {
        "unique_id": "exec-uuid-101",
        "step": "llm_call",
        "status": "completed",
        "input": "How do I reset my password?",
        "output": "To reset your password...",
        "tokens_used": 245,
        "duration_ms": 1250,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

## Data Models

### Thread
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Thread title |
| `messages_count` | integer | Number of messages |
| `runs_count` | integer | Number of runs |
| `status` | enum | active, archived |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Run
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `status` | enum | queued, running, completed, failed |
| `input_message` | string | User input |
| `output_message` | string | Agent response |
| `tokens_used` | integer | Total tokens consumed |
| `duration_ms` | integer | Execution time in ms |
| `model` | string | LLM model used |
| `created_at` | timestamp | Creation time |
| `completed_at` | timestamp | Completion time |

### Execution
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `step` | string | Execution step type |
| `status` | enum | pending, running, completed, failed |
| `input` | string | Step input |
| `output` | string | Step output |
| `tokens_used` | integer | Tokens consumed |
| `duration_ms` | integer | Step duration in ms |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Thread Not Found",
    "detail": "The requested thread could not be found."
  }]
}
```
