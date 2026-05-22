---
name: 23blocks-jarvis-tools-api
description: Manage 23blocks Jarvis global reusable tools via REST API. Use when creating, listing, updating, or deleting global tool definitions that can be assigned to agents.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Tools API

Complete API reference for 23blocks Jarvis global reusable tool management.

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

### GET /tools - List Tools

Lists all global reusable tools with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tools?page=1&records=20" \
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
      "id": "tool-uuid-123",
      "type": "tool",
      "attributes": {
        "unique_id": "tool-uuid-123",
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
        "agents_count": 5,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 12
  }
}
```

---

### GET /tools/:id - Get Tool

Retrieves a single tool by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tools/tool-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "tool-uuid-123",
    "type": "tool",
    "attributes": {
      "unique_id": "tool-uuid-123",
      "name": "web_search",
      "description": "Search the web for information",
      "tool_type": "function",
      "parameters": {
        "type": "object",
        "properties": {
          "query": { "type": "string", "description": "Search query" },
          "max_results": { "type": "integer", "description": "Maximum results to return" }
        },
        "required": ["query"]
      },
      "agents_count": 5,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Tool not found

---

### POST /tools - Create Tool

Creates a new global reusable tool.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/tools" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": {
      "name": "web_search",
      "description": "Search the web for information",
      "tool_type": "function",
      "parameters": {
        "type": "object",
        "properties": {
          "query": { "type": "string", "description": "Search query" },
          "max_results": { "type": "integer", "description": "Maximum results" }
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
    "id": "tool-uuid-123",
    "type": "tool",
    "attributes": {
      "unique_id": "tool-uuid-123",
      "name": "web_search",
      "description": "Search the web for information",
      "tool_type": "function",
      "agents_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /tools/:id - Update Tool

Updates an existing tool.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/tools/tool-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": {
      "description": "Search the web for up-to-date information",
      "parameters": {
        "type": "object",
        "properties": {
          "query": { "type": "string", "description": "Search query" },
          "max_results": { "type": "integer", "description": "Maximum results (default: 10)" },
          "language": { "type": "string", "description": "Language filter" }
        },
        "required": ["query"]
      }
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "tool-uuid-123",
    "type": "tool",
    "attributes": {
      "unique_id": "tool-uuid-123",
      "name": "web_search",
      "description": "Search the web for up-to-date information",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /tools/:id - Delete Tool

Deletes a global tool.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/tools/tool-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Tool not found

---

## Data Models

### Tool
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Tool name |
| `description` | string | Tool description |
| `tool_type` | string | Tool type (function) |
| `parameters` | object | JSON Schema for parameters |
| `agents_count` | integer | Number of agents using this tool |
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
// ToolsService — client.jarvis.tools
list(params?: ListToolsParams): Promise<PageResult<Tool>>;
get(uniqueId: string): Promise<Tool>;
create(data: CreateToolRequest): Promise<Tool>;
update(uniqueId: string, data: UpdateToolRequest): Promise<Tool>;
delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Tool,
  CreateToolRequest,
  UpdateToolRequest,
  ListToolsParams,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();
  const result = await client.jarvis.tools.list();
}
```
