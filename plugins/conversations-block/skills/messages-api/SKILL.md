---
name: conversations-messages-api
description: Send, receive, update, and manage messages within conversations in the 23blocks Conversations Block
version: 1.0.0
tags:
  - conversations
  - messages
  - drafts
  - read-receipts
env:
  - BLOCKS_API_URL
  - BLOCKS_AUTH_TOKEN
  - BLOCKS_API_KEY
---

# Messages API

Send, receive, update, and manage messages within conversations. Supports read/unread tracking, message extensions, and draft messages.

## Authentication

All requests require the following headers:

```
Authorization: Bearer $BLOCKS_AUTH_TOKEN
AppId: $BLOCKS_API_KEY
```

## Base URL

```
https://conversations.api.us.23blocks.com
```

---

## Endpoints

### Get Message

Retrieve a specific message from a conversation.

**Endpoint:** `GET /conversations/:unique_id/messages/:message_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| message_id | string | Yes | The message unique ID |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/conversations/conv_def456/messages/msg_001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "msg_001",
    "type": "messages",
    "attributes": {
      "unique_id": "msg_001",
      "conversation_id": "conv_def456",
      "sender_id": "usr_abc123",
      "content": "Hello, how are you?",
      "message_type": "text",
      "read_by": ["usr_abc123", "usr_xyz789"],
      "attachments": [],
      "status": "read",
      "extended_data": null,
      "created_at": "2025-01-15T10:30:00Z",
      "updated_at": "2025-01-15T10:30:00Z"
    }
  }
}
```

---

### Send Message

Send a new message to a conversation.

**Endpoint:** `POST /conversations/:unique_id/messages`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| content | string | Yes | Message content |
| message_type | string | No | Type: `text` (default), `image`, `file`, `system`, `rich` |
| sender_id | string | Yes | Unique ID of the sending user |
| attachments | array | No | Array of attachment objects |
| metadata | object | No | Additional message metadata |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/conversations/conv_def456/messages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Hello, how are you?",
    "message_type": "text",
    "sender_id": "usr_abc123",
    "metadata": {
      "client_id": "client_msg_001"
    }
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "msg_new001",
    "type": "messages",
    "attributes": {
      "unique_id": "msg_new001",
      "conversation_id": "conv_def456",
      "sender_id": "usr_abc123",
      "content": "Hello, how are you?",
      "message_type": "text",
      "read_by": ["usr_abc123"],
      "attachments": [],
      "status": "sent",
      "metadata": {
        "client_id": "client_msg_001"
      },
      "created_at": "2025-01-15T11:00:00Z",
      "updated_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

### Send Message with Attachments

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/conversations/conv_def456/messages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Here is the document",
    "message_type": "file",
    "sender_id": "usr_abc123",
    "attachments": [
      {
        "file_unique_id": "file_001",
        "filename": "report.pdf",
        "content_type": "application/pdf",
        "size": 245000,
        "url": "https://cdn.23blocks.com/conversations/conv_def456/files/report.pdf"
      }
    ]
  }'
```

---

### Update Message

Update an existing message in a conversation.

**Endpoint:** `PUT /conversations/:unique_id/messages`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| message_unique_id | string | Yes | The unique ID of the message to update |
| content | string | No | Updated message content |
| metadata | object | No | Updated metadata |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/conversations/conv_def456/messages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message_unique_id": "msg_001",
    "content": "Hello, how are you doing today?"
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "msg_001",
    "type": "messages",
    "attributes": {
      "unique_id": "msg_001",
      "conversation_id": "conv_def456",
      "sender_id": "usr_abc123",
      "content": "Hello, how are you doing today?",
      "message_type": "text",
      "edited": true,
      "edited_at": "2025-01-15T11:30:00Z",
      "status": "sent",
      "created_at": "2025-01-15T10:30:00Z",
      "updated_at": "2025-01-15T11:30:00Z"
    }
  }
}
```

---

### Mark Message as Read

Mark a specific message as read by the current user.

**Endpoint:** `PUT /conversations/:unique_id/messages/:message_unique_id/read`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| message_unique_id | string | Yes | The message unique ID |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/conversations/conv_def456/messages/msg_001/read" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "msg_001",
    "type": "messages",
    "attributes": {
      "unique_id": "msg_001",
      "status": "read",
      "read_by": ["usr_abc123", "usr_xyz789"],
      "read_at": "2025-01-15T11:45:00Z"
    }
  }
}
```

---

### Mark Message as Unread

Mark a specific message as unread for the current user.

**Endpoint:** `PUT /conversations/:unique_id/messages/:message_unique_id/unread`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| message_unique_id | string | Yes | The message unique ID |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/conversations/conv_def456/messages/msg_001/unread" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "msg_001",
    "type": "messages",
    "attributes": {
      "unique_id": "msg_001",
      "status": "delivered",
      "read_by": ["usr_abc123"],
      "updated_at": "2025-01-15T12:00:00Z"
    }
  }
}
```

---

### Extend Message

Attach additional custom data to a message.

**Endpoint:** `PUT /conversations/:unique_id/messages/:message_unique_id/extend`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| message_unique_id | string | Yes | The message unique ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| data | object | Yes | Custom data to attach to the message |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/conversations/conv_def456/messages/msg_001/extend" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "data": {
      "reactions": {
        "thumbs_up": ["usr_xyz789"],
        "heart": ["usr_lmn456"]
      },
      "pinned": true,
      "pinned_by": "usr_abc123"
    }
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "msg_001",
    "type": "messages",
    "attributes": {
      "unique_id": "msg_001",
      "extended_data": {
        "reactions": {
          "thumbs_up": ["usr_xyz789"],
          "heart": ["usr_lmn456"]
        },
        "pinned": true,
        "pinned_by": "usr_abc123"
      },
      "updated_at": "2025-01-15T12:30:00Z"
    }
  }
}
```

---

## Draft Messages

### Get Draft Message

Retrieve a draft message from a conversation.

**Endpoint:** `GET /conversations/:unique_id/draft_messages/:message_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| message_id | string | Yes | The draft message unique ID |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/conversations/conv_def456/draft_messages/draft_001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "draft_001",
    "type": "draft_messages",
    "attributes": {
      "unique_id": "draft_001",
      "conversation_id": "conv_def456",
      "sender_id": "usr_abc123",
      "content": "I was thinking about the project timeline...",
      "message_type": "text",
      "attachments": [],
      "metadata": {},
      "created_at": "2025-01-15T10:00:00Z",
      "updated_at": "2025-01-15T10:15:00Z"
    }
  }
}
```

---

### Create Draft Message

Save a draft message in a conversation.

**Endpoint:** `POST /conversations/:unique_id/draft_messages`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| content | string | Yes | Draft message content |
| sender_id | string | Yes | Unique ID of the sender |
| message_type | string | No | Type: `text` (default), `image`, `file`, `rich` |
| attachments | array | No | Array of draft attachment objects |
| metadata | object | No | Additional draft metadata |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/conversations/conv_def456/draft_messages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "I was thinking about the project timeline...",
    "sender_id": "usr_abc123",
    "message_type": "text"
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "draft_new001",
    "type": "draft_messages",
    "attributes": {
      "unique_id": "draft_new001",
      "conversation_id": "conv_def456",
      "sender_id": "usr_abc123",
      "content": "I was thinking about the project timeline...",
      "message_type": "text",
      "attachments": [],
      "metadata": {},
      "created_at": "2025-01-15T11:00:00Z",
      "updated_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

## Data Model

### Message

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the message |
| conversation_id | string | Parent conversation ID |
| sender_id | string | User who sent the message |
| content | string | Message body content |
| message_type | string | Type: `text`, `image`, `file`, `system`, `rich` |
| read_by | array | List of user IDs who have read the message |
| attachments | array | Array of file attachment objects |
| status | string | Delivery status: `sent`, `delivered`, `read` |
| edited | boolean | Whether the message has been edited |
| edited_at | datetime | Timestamp of last edit |
| extended_data | object | Custom data attached via /extend |
| metadata | object | Arbitrary key-value metadata |
| created_at | datetime | Message creation timestamp |
| updated_at | datetime | Last update timestamp |

### Draft Message

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the draft |
| conversation_id | string | Parent conversation ID |
| sender_id | string | User who created the draft |
| content | string | Draft message content |
| message_type | string | Type: `text`, `image`, `file`, `rich` |
| attachments | array | Array of draft attachment objects |
| metadata | object | Arbitrary key-value metadata |
| created_at | datetime | Draft creation timestamp |
| updated_at | datetime | Last update timestamp |

### Attachment

| Field | Type | Description |
|-------|------|-------------|
| file_unique_id | string | Unique identifier for the file |
| filename | string | Original filename |
| content_type | string | MIME type |
| size | integer | File size in bytes |
| url | string | CDN URL for file access |

---

## Error Responses

### 401 Unauthorized

```json
{
  "errors": [
    {
      "status": "401",
      "title": "Unauthorized",
      "detail": "Invalid or missing authentication token"
    }
  ]
}
```

### 403 Forbidden

```json
{
  "errors": [
    {
      "status": "403",
      "title": "Forbidden",
      "detail": "User is not a participant in this conversation"
    }
  ]
}
```

### 404 Not Found

```json
{
  "errors": [
    {
      "status": "404",
      "title": "Not Found",
      "detail": "Message with unique_id 'msg_invalid' not found"
    }
  ]
}
```

### 422 Unprocessable Entity

```json
{
  "errors": [
    {
      "status": "422",
      "title": "Unprocessable Entity",
      "detail": "Message content cannot be empty"
    }
  ]
}
```
