# Agents API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

## Contents

- GET `/agents`: List Agents
- GET `/agents/:id`: Get Agent
- POST `/agents`: Create Agent
- PUT `/agents/:id`: Update Agent
- DELETE `/agents/:id`: Delete Agent
- POST `/agents/:id/prompts/:prompt_id`: Add Prompt to Agent
- DELETE `/agents/:id/prompts/:prompt_id`: Remove Prompt from Agent
- POST `/agents/:id/entities/:entity_id`: Add Entity to Agent
- DELETE `/agents/:id/entities/:entity_id`: Remove Entity from Agent
- POST `/agents/:id/context/:context_id/handoff`: Create Handoff
- GET `/agents/:id/context/:context_id/handoff`: List Handoffs
- DELETE `/agents/:id/context/:context_id/handoff/:delegation_id`: Revoke Handoff

### GET /agents - List Agents

Lists all AI agents with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents?page=1&records=20" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "agent": {
      "name": "Customer Support Bot",
      "description": "Handles customer inquiries and resolves issues",
      "system_prompt": "You are a helpful customer support agent. Be polite and thorough.",
      "provider": "mistral",
      "model": "mistral-small-latest"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Agent name |
| `description` | string | No | Agent description |
| `system_prompt` | string | No | System prompt for agent behavior |
| `provider` | string | No | LLM provider: `openai` (default), `anthropic`, `google`, `mistral`, `perplexity`, `openai_compatible`, `custom` |
| `model` | string | No | Model identifier for the provider (e.g., `gpt-4`, `mistral-small-latest`) |
| `code` | string | No | Agent code/identifier |
| `supervisor_user_uid` | uuid | No | User UID to assign as agent supervisor |
| `status` | string | No | Agent status: `active`, `inactive` |

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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Entity removed from agent successfully"
}
```

---

## Supervisor Handoff

### POST /agents/:id/context/:context_id/handoff - Create Handoff

Creates a supervisor handoff delegation for an agent context. Allows the supervisor to delegate agent management to another user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/context/context-uuid-456/handoff" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "handoff": {
      "delegate_user_uid": "user-uuid-789",
      "permissions": ["read", "write", "execute"],
      "expires_at": "2025-06-01T00:00:00Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `delegate_user_uid` | uuid | Yes | User UID to delegate to |
| `access_type` | string | No | Access type for the handoff delegation |
| `expires_in_hours` | integer | No | Number of hours until delegation expires |
| `reason` | string | No | Reason for the handoff |
| `permissions` | array | No | Permissions to grant: `read`, `write`, `execute` |
| `expires_at` | timestamp | No | Delegation expiration time |

**Response 201:**
```json
{
  "data": {
    "id": "delegation-uuid-001",
    "type": "delegation",
    "attributes": {
      "unique_id": "delegation-uuid-001",
      "agent_uid": "agent-uuid-123",
      "context_uid": "context-uuid-456",
      "delegate_user_uid": "user-uuid-789",
      "permissions": ["read", "write", "execute"],
      "status": "active",
      "expires_at": "2025-06-01T00:00:00Z",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors
- `403 Forbidden` - Not the agent supervisor

---

### GET /agents/:id/context/:context_id/handoff - List Handoffs

Lists all handoff delegations for an agent context.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/context/context-uuid-456/handoff" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "delegation-uuid-001",
      "type": "delegation",
      "attributes": {
        "unique_id": "delegation-uuid-001",
        "agent_uid": "agent-uuid-123",
        "context_uid": "context-uuid-456",
        "delegate_user_uid": "user-uuid-789",
        "permissions": ["read", "write", "execute"],
        "status": "active",
        "expires_at": "2025-06-01T00:00:00Z",
        "created_at": "2025-01-12T10:30:00Z"
      }
    }
  ]
}
```

---

### DELETE /agents/:id/context/:context_id/handoff/:delegation_id - Revoke Handoff

Revokes a supervisor handoff delegation.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/agents/agent-uuid-123/context/context-uuid-456/handoff/delegation-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Delegation not found
- `403 Forbidden` - Not the agent supervisor

---
