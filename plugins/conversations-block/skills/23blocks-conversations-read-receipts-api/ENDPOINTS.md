# Read Receipts API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Endpoints

### Mark Message as Read

Mark a specific message as read for the authenticated user. Creates a `MessageReadReceipt` record.

**Endpoint:** `PUT /conversations/:unique_id/messages/:message_unique_id/read`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| message_unique_id | string | Yes | The message unique ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| message.unique_id | string | No | Alternative: pass message ID in body instead of path |
| message.screen | string | No | Screen/view where the read occurred |
| message.app | string | No | Application identifier |

**cURL Example:**

```bash
curl -s -X PUT "$BLOCKS_API_URL/conversations/conv_def456/messages/msg_001/read" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "screen": "chat_view",
      "app": "web_client"
    }
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "type": "message",
    "id": 1,
    "attributes": {
      "message_unique_id": "msg_001",
      "context_unique_id": "conv_def456",
      "user_unique_id": "usr_xyz789",
      "read_at": "2025-01-15T11:00:00Z",
      "status": "read"
    }
  }
}
```

---

### Mark Message as Unread

Mark a specific message as unread for the authenticated user. Removes the user's `MessageReadReceipt` record.

**Endpoint:** `PUT /conversations/:unique_id/messages/:message_unique_id/unread`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| message_unique_id | string | Yes | The message unique ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| message.unique_id | string | No | Alternative: pass message ID in body |
| message.screen | string | No | Screen/view identifier |
| message.app | string | No | Application identifier |

**cURL Example:**

```bash
curl -s -X PUT "$BLOCKS_API_URL/conversations/conv_def456/messages/msg_001/unread" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "unique_id": "msg_001",
      "screen": "chat_view",
      "app": "web_client"
    }
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "type": "message",
    "id": 1,
    "attributes": {
      "message_unique_id": "msg_001",
      "context_unique_id": "conv_def456",
      "user_unique_id": "usr_xyz789",
      "status": "unread"
    }
  }
}
```

---

### Mark All Messages as Read

Mark all messages in a conversation as read for the authenticated user. Updates the user's read horizon (`last_read_at` on `context_users`).

**Endpoint:** `PUT /conversations/:unique_id/messages/read_all`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |

**cURL Example:**

```bash
curl -s -X PUT "$BLOCKS_API_URL/conversations/conv_def456/messages/read_all" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response:** `204 No Content`

---

## Legacy Endpoint

The following endpoint is still supported but routes to the same `read_message` action:

| Method | Path | Description |
|--------|------|-------------|
| PUT | `/conversations/:unique_id/messages` | Mark message as read (pass `message.unique_id` in body) |

**cURL Example:**

```bash
curl -s -X PUT "$BLOCKS_API_URL/conversations/conv_def456/messages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "unique_id": "msg_001",
      "screen": "chat_view",
      "app": "web_client"
    }
  }'
```

> **Recommendation:** Prefer the explicit `/messages/:message_unique_id/read` path for clarity.
