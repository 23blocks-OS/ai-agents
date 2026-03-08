# Conversations API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

### GET /conversations - List Conversations

Lists all standalone conversations with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/conversations?page=1&records=20" \
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
      "id": "conv-uuid-123",
      "type": "conversation",
      "attributes": {
        "unique_id": "conv-uuid-123",
        "title": "Project Planning",
        "messages_count": 25,
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

### GET /conversations/:id - Get Conversation

Retrieves a single conversation by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/conversations/conv-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "conv-uuid-123",
    "type": "conversation",
    "attributes": {
      "unique_id": "conv-uuid-123",
      "title": "Project Planning",
      "messages_count": 25,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T11:45:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Conversation not found

---

### POST /conversations - Create Conversation

Creates a new standalone conversation.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/conversations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation": {
      "title": "Project Planning"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | No | Conversation title |

**Response 201:**
```json
{
  "data": {
    "id": "conv-uuid-123",
    "type": "conversation",
    "attributes": {
      "unique_id": "conv-uuid-123",
      "title": "Project Planning",
      "messages_count": 0,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /conversations/:id - Update Conversation

Updates a conversation.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/conversations/conv-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation": {
      "title": "Updated Project Planning"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "conv-uuid-123",
    "type": "conversation",
    "attributes": {
      "unique_id": "conv-uuid-123",
      "title": "Updated Project Planning",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /conversations/:id - Delete Conversation

Deletes a conversation.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/conversations/conv-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Messages

### GET /conversations/:id/messages - List Messages

Lists messages in a conversation.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/conversations/conv-uuid-123/messages?page=1&records=50" \
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
        "content": "What are the project milestones?",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "msg-uuid-102",
      "type": "message",
      "attributes": {
        "unique_id": "msg-uuid-102",
        "role": "assistant",
        "content": "The key milestones are: 1) Design phase complete by Feb 1...",
        "created_at": "2025-01-10T10:30:05Z"
      }
    }
  ]
}
```

---

### POST /conversations/:id/messages - Send Message

Sends a message in a conversation.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/conversations/conv-uuid-123/messages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "role": "user",
      "content": "What are the project milestones?"
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
      "content": "What are the project milestones?",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## AI Query

### POST /conversations/:id/query - Query Conversation

Queries the conversation history using AI.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/conversations/conv-uuid-123/query" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What decisions were made about the timeline?"
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
    "answer": "Based on the conversation, the team decided to move the deadline to March 15 and split the project into two phases.",
    "relevant_messages": [
      {
        "message_id": "msg-uuid-050",
        "content": "Let's push the deadline to March 15...",
        "relevance_score": 0.95
      }
    ]
  }
}
```

---

## Conversation Management

### PUT /conversations/:id/rename - Rename Conversation

Renames a conversation.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/conversations/conv-uuid-123/rename" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Q1 Project Planning"
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "conv-uuid-123",
    "type": "conversation",
    "attributes": {
      "unique_id": "conv-uuid-123",
      "title": "Q1 Project Planning",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### PUT /conversations/:id/archive - Archive Conversation

Archives a conversation.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/conversations/conv-uuid-123/archive" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "conv-uuid-123",
    "type": "conversation",
    "attributes": {
      "unique_id": "conv-uuid-123",
      "status": "archived",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### PUT /conversations/:id/restore - Restore Conversation

Restores an archived conversation.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/conversations/conv-uuid-123/restore" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "conv-uuid-123",
    "type": "conversation",
    "attributes": {
      "unique_id": "conv-uuid-123",
      "status": "active",
      "updated_at": "2025-01-12T15:00:00Z"
    }
  }
}
```
