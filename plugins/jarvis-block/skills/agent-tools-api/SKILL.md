---
name: jarvis-agent-tools-api
description: Manage 23blocks Jarvis agent tools via REST API. Use when managing legacy agent-specific tools or assigning global reusable tools to agents.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Agent Tools API

Complete API reference for 23blocks Jarvis agent tool management — legacy tools and global tool assignments.

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
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid/tools" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Legacy Agent Tools

### GET /agents/:id/tools - List Agent Tools

Lists all legacy tools for an agent.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/tools" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "tool-uuid-456",
      "type": "agent_tool",
      "attributes": {
        "unique_id": "tool-uuid-456",
        "name": "web_search",
        "description": "Search the web for information",
        "tool_type": "function",
        "parameters": {
          "type": "object",
          "properties": {
            "query": { "type": "string", "description": "Search query" }
          },
          "required": ["query"]
        },
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /agents/:id/tools - Create Agent Tool

Creates a legacy tool for an agent.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/tools" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": {
      "name": "web_search",
      "description": "Search the web for information",
      "tool_type": "function",
      "parameters": {
        "type": "object",
        "properties": {
          "query": { "type": "string", "description": "Search query" }
        },
        "required": ["query"]
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Tool name |
| `description` | string | Yes | Tool description |
| `tool_type` | string | Yes | Tool type (function) |
| `parameters` | object | Yes | JSON Schema for parameters |

**Response 201:**
```json
{
  "data": {
    "id": "tool-uuid-456",
    "type": "agent_tool",
    "attributes": {
      "unique_id": "tool-uuid-456",
      "name": "web_search",
      "description": "Search the web for information",
      "tool_type": "function",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /agents/:id/tools/:tool_id - Update Agent Tool

Updates a legacy agent tool.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/agents/agent-uuid-123/tools/tool-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": {
      "description": "Updated description for web search tool"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "tool-uuid-456",
    "type": "agent_tool",
    "attributes": {
      "unique_id": "tool-uuid-456",
      "name": "web_search",
      "description": "Updated description for web search tool",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /agents/:id/tools/:tool_id - Delete Agent Tool

Deletes a legacy agent tool.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/agents/agent-uuid-123/tools/tool-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Global Tool Assignments

### GET /agents/:id/global_tools - List Global Tool Assignments

Lists global tools assigned to an agent.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/global_tools" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "global-tool-uuid-789",
      "type": "tool",
      "attributes": {
        "unique_id": "global-tool-uuid-789",
        "name": "calculator",
        "description": "Perform mathematical calculations",
        "tool_type": "function",
        "assigned_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /agents/:id/global_tools/:tool_id - Assign Global Tool

Assigns a global reusable tool to an agent.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/global_tools/global-tool-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Global tool assigned to agent successfully"
}
```

---

### DELETE /agents/:id/global_tools/:tool_id - Unassign Global Tool

Removes a global tool assignment from an agent.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/agents/agent-uuid-123/global_tools/global-tool-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Global tool unassigned from agent successfully"
}
```

---

## Data Models

### AgentTool (Legacy)
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Tool name |
| `description` | string | Tool description |
| `tool_type` | string | Tool type (function) |
| `parameters` | object | JSON Schema for parameters |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Tool Not Found",
    "detail": "The requested tool could not be found."
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
// AgentToolsService — client.jarvis.agentTools
list(agentUniqueId: string, params?: ListAgentToolsParams): Promise<PageResult<AgentTool>>;
get(agentUniqueId: string, uniqueId: string): Promise<AgentTool>;
create(agentUniqueId: string, data: CreateAgentToolRequest): Promise<AgentTool>;
update(agentUniqueId: string, uniqueId: string, data: UpdateAgentToolRequest): Promise<AgentTool>;
delete(agentUniqueId: string, uniqueId: string): Promise<void>;

// AgentToolAssignmentsService — client.jarvis.agentToolAssignments
list(agentUniqueId: string, params?: ListAgentToolAssignmentsParams): Promise<PageResult<AgentToolAssignment>>;
get(agentUniqueId: string, uniqueId: string): Promise<AgentToolAssignment>;
create(agentUniqueId: string, data: CreateAgentToolAssignmentRequest): Promise<AgentToolAssignment>;
update(agentUniqueId: string, uniqueId: string, data: UpdateAgentToolAssignmentRequest): Promise<AgentToolAssignment>;
delete(agentUniqueId: string, uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  AgentTool,
  CreateAgentToolRequest,
  UpdateAgentToolRequest,
  ListAgentToolsParams,
  AgentToolAssignment,
  CreateAgentToolAssignmentRequest,
  UpdateAgentToolAssignmentRequest,
  ListAgentToolAssignmentsParams,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();

  // Example: list tools for an agent
  const tools = await client.jarvis.agentTools.list('agent-uuid');
}
```
