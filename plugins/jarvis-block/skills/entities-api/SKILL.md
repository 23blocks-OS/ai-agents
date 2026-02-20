---
name: jarvis-entities-api
description: Manage 23blocks Jarvis digital twin entities via REST API. Use when creating entities, managing entity contexts, handling entity conversations and messages, streaming responses, or querying entity files with AI.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Entities API

Complete API reference for 23blocks Jarvis digital twin entity management with contexts, conversations, streaming, and AI file queries.

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
curl -X GET "$BLOCKS_API_URL/entities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /entities - List Entities

Lists all digital twin entities with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities?page=1&records=20" \
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
      "id": "entity-uuid-123",
      "type": "entity",
      "attributes": {
        "unique_id": "entity-uuid-123",
        "name": "Product Knowledge Base",
        "entity_type": "knowledge_base",
        "description": "Contains product documentation",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 42
  }
}
```

---

### GET /entities/:id - Get Entity

Retrieves a single entity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/entity-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "entity-uuid-123",
    "type": "entity",
    "attributes": {
      "unique_id": "entity-uuid-123",
      "name": "Product Knowledge Base",
      "entity_type": "knowledge_base",
      "description": "Contains product documentation",
      "status": "active",
      "contexts_count": 5,
      "conversations_count": 18,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Entity not found

---

### POST /entities - Create Entity

Creates a new digital twin entity.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "entity": {
      "name": "Product Knowledge Base",
      "entity_type": "knowledge_base",
      "description": "Contains product documentation and FAQs"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Entity name |
| `entity_type` | string | Yes | Type classification |
| `description` | string | No | Entity description |

**Response 201:**
```json
{
  "data": {
    "id": "entity-uuid-123",
    "type": "entity",
    "attributes": {
      "unique_id": "entity-uuid-123",
      "name": "Product Knowledge Base",
      "entity_type": "knowledge_base",
      "description": "Contains product documentation and FAQs",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /entities/:id - Update Entity

Updates an existing entity.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/entities/entity-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "entity": {
      "name": "Updated Knowledge Base",
      "description": "Updated description with new scope"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "entity-uuid-123",
    "type": "entity",
    "attributes": {
      "unique_id": "entity-uuid-123",
      "name": "Updated Knowledge Base",
      "description": "Updated description with new scope",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /entities/:id - Delete Entity

Deletes an entity.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/entities/entity-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Entity not found

---

## Entity Contexts

### GET /entities/:id/contexts - List Entity Contexts

Retrieves all contexts for an entity.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/entity-uuid-123/contexts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "context-uuid-456",
      "type": "context",
      "attributes": {
        "unique_id": "context-uuid-456",
        "name": "Product Specs",
        "content": "Detailed product specifications and features",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /entities/:id/contexts - Create Entity Context

Creates a new context for an entity.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/entity-uuid-123/contexts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "context": {
      "name": "Product Specs",
      "content": "The product supports features X, Y, and Z. Pricing starts at $99/month."
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Context name |
| `content` | string | Yes | Context content/instructions |

**Response 201:**
```json
{
  "data": {
    "id": "context-uuid-456",
    "type": "context",
    "attributes": {
      "unique_id": "context-uuid-456",
      "name": "Product Specs",
      "content": "The product supports features X, Y, and Z. Pricing starts at $99/month.",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Entity Conversations

### GET /entities/:id/conversations - List Entity Conversations

Lists all conversations for an entity.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/entity-uuid-123/conversations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "conv-uuid-789",
      "type": "conversation",
      "attributes": {
        "unique_id": "conv-uuid-789",
        "title": "Product Q&A",
        "messages_count": 24,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 18
  }
}
```

---

### POST /entities/:id/conversations - Create Entity Conversation

Creates a new conversation for an entity.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/entity-uuid-123/conversations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation": {
      "title": "Product Q&A"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "conv-uuid-789",
    "type": "conversation",
    "attributes": {
      "unique_id": "conv-uuid-789",
      "title": "Product Q&A",
      "messages_count": 0,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### GET /entities/:id/conversations/:conv_id/messages - List Entity Messages

Lists messages in an entity conversation.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/entity-uuid-123/conversations/conv-uuid-789/messages" \
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
        "content": "What features does the premium plan include?",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /entities/:id/conversations/:conv_id/messages - Send Entity Message

Sends a message in an entity conversation.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/entity-uuid-123/conversations/conv-uuid-789/messages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "role": "user",
      "content": "What features does the premium plan include?"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "msg-uuid-101",
    "type": "message",
    "attributes": {
      "unique_id": "msg-uuid-101",
      "role": "user",
      "content": "What features does the premium plan include?",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /entities/:id/conversations/:conv_id/messages/stream - Stream Entity Message

Sends a message and streams the AI response in real-time.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/entity-uuid-123/conversations/conv-uuid-789/messages/stream" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "content": "Explain the premium plan features in detail."
    }
  }'
```

**Response 200 (Server-Sent Events):**
```
data: {"type":"token","content":"The"}
data: {"type":"token","content":" premium"}
data: {"type":"token","content":" plan"}
data: {"type":"token","content":" includes"}
data: {"type":"done","message_id":"msg-uuid-102"}
```

---

## File Query

### POST /entities/:id/file_query - Query Entity Files with AI

Queries entity-associated files using AI-powered search (RAG).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/entity-uuid-123/file_query" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What are the system requirements for installation?"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Natural language query |

**Response 200:**
```json
{
  "data": {
    "answer": "The system requirements include 8GB RAM, 50GB disk space, and a modern web browser.",
    "sources": [
      {
        "file_id": "file-uuid-789",
        "file_name": "installation-guide.pdf",
        "relevance_score": 0.95,
        "excerpt": "Minimum requirements: 8GB RAM..."
      }
    ]
  }
}
```

---

## Data Models

### Entity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Entity name |
| `entity_type` | string | Type classification |
| `description` | string | Entity description |
| `status` | enum | active, inactive |
| `contexts_count` | integer | Number of contexts |
| `conversations_count` | integer | Number of conversations |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Context
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Context name |
| `content` | string | Context content |
| `created_at` | timestamp | Creation time |

### Message
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `role` | enum | user, assistant, system |
| `content` | string | Message content |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Entity Not Found",
    "detail": "The requested entity could not be found."
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
// EntitiesService — client.jarvis.entities
list(params?: ListEntitiesParams): Promise<PageResult<Entity>>;
get(uniqueId: string): Promise<Entity>;
register(uniqueId: string, data?: RegisterEntityRequest): Promise<Entity>;
update(uniqueId: string, data: UpdateEntityRequest): Promise<Entity>;
delete(uniqueId: string): Promise<void>;
addPrompt(uniqueId: string, promptUniqueId: string): Promise<void>;
createContext(uniqueId: string, data?: CreateContextRequest): Promise<unknown>;
sendMessage(uniqueId: string, contextUniqueId: string, data: SendMessageRequest): Promise<unknown>;
sendMessageStream(uniqueId: string, contextUniqueId: string, data: SendMessageRequest): Promise<ReadableStream<string>>;
```

### TypeScript Types

```typescript
import type {
  Entity,
  RegisterEntityRequest,
  UpdateEntityRequest,
  ListEntitiesParams,
  CreateContextRequest,
  SendMessageRequest,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();
  const result = await client.jarvis.entities.list();
}
```
