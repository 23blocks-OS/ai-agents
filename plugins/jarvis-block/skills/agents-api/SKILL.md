---
name: jarvis-agents-api
description: Manage 23blocks Jarvis AI agents via REST API. Use when creating agents, configuring agent settings, assigning prompts to agents, or binding entities to agents.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Agents API

Complete API reference for 23blocks Jarvis AI agent management with prompt assignments and entity bindings.

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
curl -X GET "$BLOCKS_API_URL/agents" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /agents - List Agents

Lists all AI agents with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents?page=1&records=20" \
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
      "id": "agent-uuid-123",
      "type": "agent",
      "attributes": {
        "unique_id": "agent-uuid-123",
        "name": "Customer Support Bot",
        "description": "Handles customer inquiries",
        "system_prompt": "You are a helpful customer support agent.",
        "status": "active",
        "prompts_count": 3,
        "entities_count": 2,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 35
  }
}
```

---

### GET /agents/:id - Get Agent

Retrieves a single agent by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "agent-uuid-123",
    "type": "agent",
    "attributes": {
      "unique_id": "agent-uuid-123",
      "name": "Customer Support Bot",
      "description": "Handles customer inquiries",
      "system_prompt": "You are a helpful customer support agent.",
      "status": "active",
      "prompts_count": 3,
      "entities_count": 2,
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "prompts": {
        "data": [
          { "id": "prompt-uuid-1", "type": "prompt" }
        ]
      },
      "entities": {
        "data": [
          { "id": "entity-uuid-1", "type": "entity" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Agent not found

---

### POST /agents - Create Agent

Creates a new AI agent.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "agent": {
      "name": "Customer Support Bot",
      "description": "Handles customer inquiries and resolves issues",
      "system_prompt": "You are a helpful customer support agent. Be polite and thorough."
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Agent name |
| `description` | string | No | Agent description |
| `system_prompt` | string | No | System prompt for agent behavior |

**Response 201:**
```json
{
  "data": {
    "id": "agent-uuid-123",
    "type": "agent",
    "attributes": {
      "unique_id": "agent-uuid-123",
      "name": "Customer Support Bot",
      "description": "Handles customer inquiries and resolves issues",
      "system_prompt": "You are a helpful customer support agent. Be polite and thorough.",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /agents/:id - Update Agent

Updates an existing agent.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/agents/agent-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "agent": {
      "name": "Updated Support Bot",
      "system_prompt": "You are an expert support agent specializing in billing."
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "agent-uuid-123",
    "type": "agent",
    "attributes": {
      "unique_id": "agent-uuid-123",
      "name": "Updated Support Bot",
      "system_prompt": "You are an expert support agent specializing in billing.",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /agents/:id - Delete Agent

Deletes an agent.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/agents/agent-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Agent not found

---

## Agent Prompts

### POST /agents/:id/prompts/:prompt_id - Add Prompt to Agent

Assigns a prompt to an agent.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/prompts/prompt-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt added to agent successfully"
}
```

---

### DELETE /agents/:id/prompts/:prompt_id - Remove Prompt from Agent

Removes a prompt assignment from an agent.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/agents/agent-uuid-123/prompts/prompt-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt removed from agent successfully"
}
```

---

## Agent Entities

### POST /agents/:id/entities/:entity_id - Add Entity to Agent

Binds a digital twin entity to an agent.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/entities/entity-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Entity added to agent successfully"
}
```

---

### DELETE /agents/:id/entities/:entity_id - Remove Entity from Agent

Unbinds an entity from an agent.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/agents/agent-uuid-123/entities/entity-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Entity removed from agent successfully"
}
```

---

## Data Models

### Agent
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Agent name |
| `description` | string | Agent description |
| `system_prompt` | string | System prompt for behavior |
| `status` | enum | active, inactive |
| `prompts_count` | integer | Number of assigned prompts |
| `entities_count` | integer | Number of bound entities |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Agent Not Found",
    "detail": "The requested agent could not be found."
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
// AgentsService — client.jarvis.agents
list(params?: ListAgentsParams): Promise<PageResult<Agent>>;
get(uniqueId: string): Promise<Agent>;
create(data: CreateAgentRequest): Promise<Agent>;
update(uniqueId: string, data: UpdateAgentRequest): Promise<Agent>;
delete(uniqueId: string): Promise<void>;
addPrompt(uniqueId: string, data: AddAgentPromptRequest): Promise<Agent>;
addEntity(uniqueId: string, data: AddAgentEntityRequest): Promise<Agent>;
removeEntity(uniqueId: string, data: AddAgentEntityRequest): Promise<Agent>;
```

### TypeScript Types

```typescript
import type {
  Agent,
  CreateAgentRequest,
  UpdateAgentRequest,
  ListAgentsParams,
  AddAgentPromptRequest,
  AddAgentEntityRequest,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();

  // Example: list all agents
  const result = await client.jarvis.agents.list();
}
```
