---
name: 23blocks-jarvis-tools-api
description: "Jarvis global tool definitions: CRUD. Use when defining a reusable tool; attaching it is 23blocks-jarvis-agent-tools-api."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Tools API

Complete API reference for 23blocks Jarvis global reusable tool management.

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Tool Not Found","detail":"The requested tool could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useJarvisBlock()` from `@23blocks/react`.

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
