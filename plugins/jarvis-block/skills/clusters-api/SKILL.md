---
name: jarvis-clusters-api
description: Manage 23blocks Jarvis entity clusters via REST API. Use when grouping entities, managing cluster members, assigning prompts, or handling cluster contexts and conversations.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Clusters API

Complete API reference for 23blocks Jarvis entity cluster management with members, prompts, contexts, and conversations.

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
curl -X GET "$BLOCKS_API_URL/clusters" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /clusters - List Clusters

Lists all entity clusters with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/clusters?page=1&records=20" \
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
      "id": "cluster-uuid-123",
      "type": "cluster",
      "attributes": {
        "unique_id": "cluster-uuid-123",
        "name": "Sales Team Entities",
        "description": "Digital twins for the sales department",
        "members_count": 8,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 15
  }
}
```

---

### GET /clusters/:id - Get Cluster

Retrieves a single cluster by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/clusters/cluster-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "cluster-uuid-123",
    "type": "cluster",
    "attributes": {
      "unique_id": "cluster-uuid-123",
      "name": "Sales Team Entities",
      "description": "Digital twins for the sales department",
      "members_count": 8,
      "prompts_count": 3,
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "members": {
        "data": [
          { "id": "entity-uuid-1", "type": "entity" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Cluster not found

---

### POST /clusters - Create Cluster

Creates a new entity cluster.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/clusters" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "cluster": {
      "name": "Sales Team Entities",
      "description": "Digital twins for the sales department"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Cluster name |
| `description` | string | No | Cluster description |

**Response 201:**
```json
{
  "data": {
    "id": "cluster-uuid-123",
    "type": "cluster",
    "attributes": {
      "unique_id": "cluster-uuid-123",
      "name": "Sales Team Entities",
      "description": "Digital twins for the sales department",
      "members_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /clusters/:id - Update Cluster

Updates an existing cluster.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/clusters/cluster-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "cluster": {
      "name": "Updated Sales Team",
      "description": "Updated description"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "cluster-uuid-123",
    "type": "cluster",
    "attributes": {
      "unique_id": "cluster-uuid-123",
      "name": "Updated Sales Team",
      "description": "Updated description",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /clusters/:id - Delete Cluster

Deletes a cluster.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/clusters/cluster-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Cluster Members

### POST /clusters/:id/members/:entity_id - Add Member

Adds an entity to the cluster.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/clusters/cluster-uuid-123/members/entity-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Entity added to cluster successfully"
}
```

---

### DELETE /clusters/:id/members/:entity_id - Remove Member

Removes an entity from the cluster.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/clusters/cluster-uuid-123/members/entity-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Entity removed from cluster successfully"
}
```

---

## Cluster Prompts

### GET /clusters/:id/prompts - List Cluster Prompts

Lists prompts assigned to the cluster.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/clusters/cluster-uuid-123/prompts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "prompt-uuid-789",
      "type": "prompt",
      "attributes": {
        "unique_id": "prompt-uuid-789",
        "name": "Sales Pitch Generator",
        "description": "Generates sales pitches"
      }
    }
  ]
}
```

---

### POST /clusters/:id/prompts/:prompt_id - Add Prompt to Cluster

Assigns a prompt to the cluster.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/clusters/cluster-uuid-123/prompts/prompt-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt added to cluster successfully"
}
```

---

### DELETE /clusters/:id/prompts/:prompt_id - Remove Prompt from Cluster

Removes a prompt from the cluster.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/clusters/cluster-uuid-123/prompts/prompt-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt removed from cluster successfully"
}
```

---

## Cluster Contexts

### GET /clusters/:id/contexts - List Cluster Contexts

Retrieves all contexts for a cluster.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/clusters/cluster-uuid-123/contexts" \
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
        "name": "Sales Guidelines",
        "content": "Follow the company sales methodology",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /clusters/:id/contexts - Create Cluster Context

Creates a new context for a cluster.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/clusters/cluster-uuid-123/contexts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "context": {
      "name": "Sales Guidelines",
      "content": "Follow the company sales methodology. Always be consultative."
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "context-uuid-456",
    "type": "context",
    "attributes": {
      "unique_id": "context-uuid-456",
      "name": "Sales Guidelines",
      "content": "Follow the company sales methodology. Always be consultative.",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Cluster Conversations

### GET /clusters/:id/conversations - List Cluster Conversations

Lists all conversations for a cluster.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/clusters/cluster-uuid-123/conversations" \
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
        "title": "Team Strategy Session",
        "messages_count": 30,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /clusters/:id/conversations - Create Cluster Conversation

Creates a new conversation for a cluster.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/clusters/cluster-uuid-123/conversations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation": {
      "title": "Team Strategy Session"
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
      "title": "Team Strategy Session",
      "messages_count": 0,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### GET /clusters/:id/conversations/:conv_id/messages - List Cluster Messages

Lists messages in a cluster conversation.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/clusters/cluster-uuid-123/conversations/conv-uuid-789/messages" \
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
        "content": "What is our Q1 sales target?",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /clusters/:id/conversations/:conv_id/messages - Send Cluster Message

Sends a message in a cluster conversation.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/clusters/cluster-uuid-123/conversations/conv-uuid-789/messages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "role": "user",
      "content": "What is our Q1 sales target?"
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
      "content": "What is our Q1 sales target?",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Data Models

### Cluster
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Cluster name |
| `description` | string | Cluster description |
| `members_count` | integer | Number of member entities |
| `prompts_count` | integer | Number of assigned prompts |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Cluster Not Found",
    "detail": "The requested cluster could not be found."
  }]
}
```
