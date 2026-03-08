# Entities API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

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
