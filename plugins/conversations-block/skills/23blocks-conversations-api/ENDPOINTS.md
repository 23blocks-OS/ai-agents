# Conversations API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

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
| sort | string | No | Sort as `column:direction`. Allowed columns: `updated_at`, `created_at`, `reference`, `name`, `new_messages`. Default: `updated_at:desc` |
| has_unread | boolean | No | Filter to conversations with unread messages for this user (per-user via `context_users.unread_count`) |
| min_new_messages | integer | No | Filter to conversations with at least N new messages for this user (per-user) |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/conversations?page=1&per_page=25&status=active&sort=new_messages:desc&has_unread=true" \
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
| search | string | No | Search by conversation name (ILIKE match on `context.name`) |
| sort | string | No | Sort as `column:direction`. Allowed columns: `updated_at`, `created_at`. Default: `updated_at:desc` |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/mygroups/conversations?page=1&per_page=25&search=engineering&sort=created_at:asc" \
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

### Get Unread Summary

Retrieve aggregated per-user unread counts grouped by a dimension. Admin-only endpoint requiring `conversations:read` scope.

**Endpoint:** `GET /users/:unique_id/unread-summary`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the user |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_by | string | No | Grouping dimension: `reference` (default), `source`, `source_type`, `source_alias`, or `payload:<key>` for custom payload key grouping. Invalid fixed-column values fall back to `reference` silently. Invalid payload keys return `422` |
| custom[<key>] | string | No | Filter by conversation payload key before grouping. Composable with `group_by=payload:<key>` for multi-level drill-down (e.g., `custom[project_id]=P1&group_by=payload:role`) |

**cURL Example (fixed-column grouping):**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/unread-summary?group_by=reference" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "type": "UnreadSummary",
    "attributes": {
      "user_unique_id": "usr_abc123",
      "group_by": "reference",
      "buckets": [
        {
          "key": "project-alpha",
          "unread_count": 12,
          "conversation_count": 3
        },
        {
          "key": "support-tickets",
          "unread_count": 5,
          "conversation_count": 2
        }
      ],
      "total_unread_count": 17,
      "total_conversation_count": 5
    }
  }
}
```

**cURL Example (payload grouping):**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/unread-summary?group_by=payload:project_id" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "type": "UnreadSummary",
    "attributes": {
      "group_by": "payload:project_id",
      "buckets": [
        {
          "key": "P1",
          "unread_count": 7,
          "conversation_count": 2,
          "conversation_payload": {
            "project_id": "P1",
            "project_name": "Adventure Quest",
            "role": "Developer"
          }
        },
        {
          "key": "P2",
          "unread_count": 3,
          "conversation_count": 1,
          "conversation_payload": {
            "project_id": "P2",
            "project_name": "Project Alpha",
            "role": "Designer"
          }
        }
      ],
      "total_unread_count": 10,
      "total_conversation_count": 3
    }
  }
}
```

**cURL Example (multi-level drill-down with custom filter + payload grouping):**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/unread-summary?custom[project_id]=P1&group_by=payload:role" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Payload Grouping Notes:**
- Payload key must match `/[a-zA-Z0-9_]+/` — invalid keys return `422` with code `rt-076540`
- `conversation_payload` is only present in buckets when using `payload:<key>` grouping, not fixed-column grouping
- Each bucket's `conversation_payload` contains the full payload from one conversation in that group
- Conversations with `NULL` values for the grouped key are excluded from results

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

## AI Summaries

### Generate Conversation Summary

Generate an AI-powered summary of a conversation via Jarvis. Uses incremental processing — only sends new messages since the last summary, with the previous summary passed as context.

**Endpoint:** `POST /conversations/:unique_id/summary`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| prompt_id | string | No | Jarvis prompt ID for custom summary types (default: `conversation-summary`) |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/conversations/conv_def456/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt_id": "conversation-summary"
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "summary_001",
    "type": "conversation_summary",
    "attributes": {
      "unique_id": "summary_001",
      "conversation_id": "conv_def456",
      "summary": "The team discussed the Q2 roadmap, focusing on three key deliverables...",
      "key_points": [
        "Q2 roadmap finalized with 3 deliverables",
        "Design review scheduled for next Tuesday",
        "Budget approved for new tooling"
      ],
      "action_items": [
        "Alice to prepare design mockups by Monday",
        "Bob to set up staging environment",
        "Charlie to send budget breakdown to finance"
      ],
      "summary_type": "conversation",
      "prompt_id": "conversation-summary",
      "message_count": 47,
      "last_processed_message_id": "msg_xyz789",
      "tokens_used": 1250,
      "created_at": "2025-01-15T15:00:00Z"
    }
  }
}
```

**Rate Limiting:** 1 Jarvis call per 60 seconds per conversation per user.

**Caching:** Summaries are cached in `conversation_summaries` table with staleness detection. Subsequent calls return the cached summary if no new messages exist.

**Errors:**
- `429 Too Many Requests` - Rate limit exceeded
- `404 Not Found` - Conversation not found

---

### Generate Batch Digest

Batch-generate AI summaries for multiple conversations. Useful for inbox-level AI overviews. Only stale or missing summaries trigger Jarvis calls.

**Endpoint:** `POST /conversations/digest`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| conversation_ids | array | Yes | Array of conversation unique IDs (max 50) |
| prompt_id | string | No | Jarvis prompt ID for custom summary types (default: `conversation-summary`) |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/conversations/digest" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_ids": ["conv_def456", "conv_abc123", "conv_xyz789"]
  }'
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "summary_001",
      "type": "conversation_summary",
      "attributes": {
        "unique_id": "summary_001",
        "conversation_id": "conv_def456",
        "summary": "Q2 roadmap discussion with 3 deliverables finalized...",
        "key_points": ["Q2 roadmap finalized", "Design review Tuesday"],
        "action_items": ["Alice: mockups by Monday"],
        "summary_type": "digest",
        "message_count": 47,
        "last_processed_message_id": "msg_xyz789",
        "tokens_used": 1250,
        "created_at": "2025-01-15T15:00:00Z"
      }
    },
    {
      "id": "summary_002",
      "type": "conversation_summary",
      "attributes": {
        "unique_id": "summary_002",
        "conversation_id": "conv_abc123",
        "summary": "Bug triage session — 5 critical issues identified...",
        "key_points": ["5 critical bugs identified", "Hotfix branch created"],
        "action_items": ["Team to prioritize P0 fixes"],
        "summary_type": "digest",
        "message_count": 23,
        "last_processed_message_id": "msg_abc456",
        "tokens_used": 800,
        "created_at": "2025-01-15T15:00:00Z"
      }
    }
  ]
}
```

**Notes:**
- Maximum 50 conversations per request
- Uses 2-query batch cache check — only stale/missing summaries trigger Jarvis calls
- Cached summaries are returned immediately without a Jarvis call
