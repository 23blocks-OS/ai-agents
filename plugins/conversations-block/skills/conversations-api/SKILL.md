---
name: conversations-conversations-api
description: Create and manage conversations, file uploads, and presigned URLs in the 23blocks Conversations Block
version: 1.0.0
tags:
  - conversations
  - messaging
  - files
  - archive
env:
  - BLOCKS_API_URL
  - BLOCKS_AUTH_TOKEN
  - BLOCKS_API_KEY
---

# Conversations API

Create and manage conversations between users. Supports metadata management, archiving, restoring, file uploads, and presigned URLs for direct file uploads.

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

### List User Conversations

Retrieve all conversations for a specific user.

**Endpoint:** `GET /users/:unique_id/conversations`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the user |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number for pagination |
| per_page | integer | No | Results per page (default: 25) |
| status | string | No | Filter by status: `active`, `archived` |
| sort | string | No | Sort field (default: `-updated_at`) |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/conversations?page=1&per_page=25&status=active" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "conv_def456",
      "type": "conversations",
      "attributes": {
        "unique_id": "conv_def456",
        "participants": ["usr_abc123", "usr_xyz789"],
        "last_message": {
          "content": "Hello there!",
          "sender_id": "usr_xyz789",
          "sent_at": "2025-01-15T10:30:00Z"
        },
        "unread_count": 3,
        "metadata": {},
        "status": "active",
        "created_at": "2025-01-10T08:00:00Z",
        "updated_at": "2025-01-15T10:30:00Z"
      }
    }
  ],
  "meta": {
    "total": 12,
    "page": 1,
    "per_page": 25
  }
}
```

---

### List Group Conversations

Retrieve all group conversations for a specific user.

**Endpoint:** `GET /users/:unique_id/mygroups/conversations`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the user |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number for pagination |
| per_page | integer | No | Results per page (default: 25) |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/mygroups/conversations?page=1&per_page=25" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "conv_grp789",
      "type": "conversations",
      "attributes": {
        "unique_id": "conv_grp789",
        "group_id": "grp_001",
        "participants": ["usr_abc123", "usr_xyz789", "usr_lmn456"],
        "last_message": {
          "content": "Meeting at 3pm",
          "sender_id": "usr_lmn456",
          "sent_at": "2025-01-15T09:00:00Z"
        },
        "unread_count": 1,
        "metadata": {},
        "status": "active",
        "created_at": "2025-01-08T12:00:00Z",
        "updated_at": "2025-01-15T09:00:00Z"
      }
    }
  ],
  "meta": {
    "total": 5,
    "page": 1,
    "per_page": 25
  }
}
```

---

### Get Conversation

Retrieve details of a specific conversation.

**Endpoint:** `GET /conversations/:unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the conversation |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/conversations/conv_def456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "conv_def456",
    "type": "conversations",
    "attributes": {
      "unique_id": "conv_def456",
      "participants": ["usr_abc123", "usr_xyz789"],
      "last_message": {
        "content": "Hello there!",
        "sender_id": "usr_xyz789",
        "sent_at": "2025-01-15T10:30:00Z"
      },
      "unread_count": 0,
      "metadata": {
        "topic": "Project Discussion"
      },
      "status": "active",
      "created_at": "2025-01-10T08:00:00Z",
      "updated_at": "2025-01-15T10:30:00Z"
    }
  }
}
```

---

### Create Conversation

Create a new conversation between participants.

**Endpoint:** `POST /conversations/`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| participants | array | Yes | Array of user unique_ids |
| metadata | object | No | Initial conversation metadata |
| group_id | string | No | Associated group ID for group conversations |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/conversations/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "participants": ["usr_abc123", "usr_xyz789"],
    "metadata": {
      "topic": "Project Discussion"
    }
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "conv_new001",
    "type": "conversations",
    "attributes": {
      "unique_id": "conv_new001",
      "participants": ["usr_abc123", "usr_xyz789"],
      "last_message": null,
      "unread_count": 0,
      "metadata": {
        "topic": "Project Discussion"
      },
      "status": "active",
      "created_at": "2025-01-15T11:00:00Z",
      "updated_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

### Update Conversation Metadata

Update the metadata of an existing conversation.

**Endpoint:** `PUT /conversations/:unique_id/meta`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the conversation |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| metadata | object | Yes | Updated metadata key-value pairs |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/conversations/conv_def456/meta" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "metadata": {
      "topic": "Updated Project Discussion",
      "priority": "high"
    }
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "conv_def456",
    "type": "conversations",
    "attributes": {
      "unique_id": "conv_def456",
      "metadata": {
        "topic": "Updated Project Discussion",
        "priority": "high"
      },
      "updated_at": "2025-01-15T12:00:00Z"
    }
  }
}
```

---

### Archive Conversation

Archive a conversation to remove it from the active list.

**Endpoint:** `PUT /conversations/:unique_id/archive`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the conversation |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/conversations/conv_def456/archive" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "conv_def456",
    "type": "conversations",
    "attributes": {
      "unique_id": "conv_def456",
      "status": "archived",
      "updated_at": "2025-01-15T12:30:00Z"
    }
  }
}
```

---

### Restore Conversation

Restore an archived conversation back to active status.

**Endpoint:** `PUT /conversations/:unique_id/restore`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the conversation |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/conversations/conv_def456/restore" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "conv_def456",
    "type": "conversations",
    "attributes": {
      "unique_id": "conv_def456",
      "status": "active",
      "updated_at": "2025-01-15T13:00:00Z"
    }
  }
}
```

---

### Extend Conversation

Extend a conversation with additional custom data.

**Endpoint:** `PUT /conversations/:unique_id/extend`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the conversation |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| data | object | Yes | Custom data to attach to the conversation |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/conversations/conv_def456/extend" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "data": {
      "project_id": "proj_001",
      "department": "engineering",
      "tags": ["urgent", "review"]
    }
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "conv_def456",
    "type": "conversations",
    "attributes": {
      "unique_id": "conv_def456",
      "extended_data": {
        "project_id": "proj_001",
        "department": "engineering",
        "tags": ["urgent", "review"]
      },
      "updated_at": "2025-01-15T13:30:00Z"
    }
  }
}
```

---

## File Operations

### Get File

Retrieve a file attached to a conversation.

**Endpoint:** `GET /conversations/:unique_id/files/:file_unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| file_unique_id | string | Yes | The file unique ID |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/conversations/conv_def456/files/file_001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "file_001",
    "type": "files",
    "attributes": {
      "unique_id": "file_001",
      "conversation_id": "conv_def456",
      "filename": "report.pdf",
      "content_type": "application/pdf",
      "size": 245000,
      "url": "https://cdn.23blocks.com/conversations/conv_def456/files/report.pdf",
      "uploaded_by": "usr_abc123",
      "created_at": "2025-01-15T10:00:00Z"
    }
  }
}
```

---

### Presign File Upload

Generate a presigned URL for direct file upload.

**Endpoint:** `PUT /conversations/:unique_id/presign`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| filename | string | Yes | Name of the file to upload |
| content_type | string | Yes | MIME type of the file |
| size | integer | No | File size in bytes |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/conversations/conv_def456/presign" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "document.pdf",
    "content_type": "application/pdf",
    "size": 512000
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "presign_001",
    "type": "presigned_urls",
    "attributes": {
      "upload_url": "https://storage.23blocks.com/upload?token=abc123&expires=1705312200",
      "file_unique_id": "file_new001",
      "filename": "document.pdf",
      "content_type": "application/pdf",
      "expires_at": "2025-01-15T11:30:00Z"
    }
  }
}
```

---

### Upload File

Upload a file to a conversation.

**Endpoint:** `POST /conversations/:unique_id/files`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |

**Request Body (multipart/form-data):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| file | file | Yes | The file to upload |
| metadata | string | No | JSON string of file metadata |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/conversations/conv_def456/files" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -F "file=@/path/to/document.pdf" \
  -F 'metadata={"description": "Project report"}'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "file_new002",
    "type": "files",
    "attributes": {
      "unique_id": "file_new002",
      "conversation_id": "conv_def456",
      "filename": "document.pdf",
      "content_type": "application/pdf",
      "size": 512000,
      "url": "https://cdn.23blocks.com/conversations/conv_def456/files/document.pdf",
      "uploaded_by": "usr_abc123",
      "metadata": {
        "description": "Project report"
      },
      "created_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

### Delete File

Delete a file from a conversation.

**Endpoint:** `DELETE /conversations/:unique_id/files/:file_unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| file_unique_id | string | Yes | The file unique ID |

**cURL Example:**

```bash
curl -s -X DELETE "https://conversations.api.us.23blocks.com/conversations/conv_def456/files/file_001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "file_001",
    "type": "files",
    "attributes": {
      "unique_id": "file_001",
      "deleted": true,
      "deleted_at": "2025-01-15T14:00:00Z"
    }
  }
}
```

---

## Data Model

### Conversation

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the conversation |
| participants | array | List of user unique_ids in the conversation |
| last_message | object | Summary of the most recent message |
| unread_count | integer | Number of unread messages for the requesting user |
| metadata | object | Arbitrary key-value metadata |
| extended_data | object | Custom extended data attached via /extend |
| group_id | string | Associated group ID (null for direct conversations) |
| status | string | Conversation status: `active`, `archived` |
| created_at | datetime | Conversation creation timestamp |
| updated_at | datetime | Last update timestamp |

### File

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the file |
| conversation_id | string | Parent conversation ID |
| filename | string | Original filename |
| content_type | string | MIME type |
| size | integer | File size in bytes |
| url | string | CDN URL for file access |
| uploaded_by | string | User who uploaded the file |
| metadata | object | File metadata |
| created_at | datetime | Upload timestamp |

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

### 404 Not Found

```json
{
  "errors": [
    {
      "status": "404",
      "title": "Not Found",
      "detail": "Conversation with unique_id 'conv_invalid' not found"
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
      "detail": "Participants array must contain at least 2 user IDs"
    }
  ]
}
```

### 413 Payload Too Large

```json
{
  "errors": [
    {
      "status": "413",
      "title": "Payload Too Large",
      "detail": "File size exceeds maximum allowed size of 50MB"
    }
  ]
}
```
