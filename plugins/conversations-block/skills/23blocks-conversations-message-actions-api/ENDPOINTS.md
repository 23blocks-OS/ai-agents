# Message Actions API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Creating Actions (Inline with Message)

Actions are created as part of message creation. Pass an `actions` array when sending a message via `POST /conversations/:unique_id/messages`.

**cURL Example:**

```bash
curl -s -X POST "$BLOCKS_API_URL/conversations/conv_def456/messages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "source": "usr_abc123",
      "content": "Would you like to approve this request?",
      "actions": [
        {
          "action_type": "button",
          "control_type": "submit",
          "control_label": "Approve",
          "control_value": "approved",
          "control_class": "btn-success",
          "payload": { "request_id": "req_001" }
        },
        {
          "action_type": "button",
          "control_type": "cancel",
          "control_label": "Reject",
          "control_value": "rejected",
          "control_class": "btn-danger",
          "payload": { "request_id": "req_001" }
        }
      ]
    }
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "msg_001",
    "type": "messages",
    "attributes": {
      "unique_id": "msg_001",
      "content": "Would you like to approve this request?"
    },
    "relationships": {
      "message_actions": {
        "data": [
          { "id": "act_001", "type": "message_actions" },
          { "id": "act_002", "type": "message_actions" }
        ]
      }
    }
  },
  "included": [
    {
      "id": "act_001",
      "type": "message_actions",
      "attributes": {
        "unique_id": "act_001",
        "action_type": "button",
        "control_type": "submit",
        "control_label": "Approve",
        "control_value": "approved",
        "status": "pending"
      }
    }
  ]
}
```

---

## Endpoints

### List Message Actions

Retrieve all active actions for a message. Paginated.

**Endpoint:** `GET /conversations/:unique_id/messages/:message_unique_id/actions`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| message_unique_id | string | Yes | The message unique ID |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number (default: 1) |
| records | integer | No | Records per page (default: 50, max: 100) |

**cURL Example:**

```bash
curl -s -X GET "$BLOCKS_API_URL/conversations/conv_def456/messages/msg_001/actions?page=1&records=50" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "act_001",
      "type": "message_actions",
      "attributes": {
        "unique_id": "act_001",
        "action_type": "button",
        "control_type": "submit",
        "control_label": "Approve",
        "control_value": "approved",
        "control_class": "btn-success",
        "control_set_id": null,
        "payload": { "request_id": "req_001" },
        "response": null,
        "status": "pending",
        "enabled": "true",
        "user_unique_id": null,
        "created_at": "2025-01-15T10:00:00Z",
        "updated_at": "2025-01-15T10:00:00Z"
      },
      "relationships": {
        "message": {
          "data": { "id": "msg_001", "type": "messages" }
        }
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 2
  }
}
```

---

### Get Action

Retrieve a specific action by unique ID.

**Endpoint:** `GET /conversations/:unique_id/messages/:message_unique_id/actions/:action_unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| message_unique_id | string | Yes | The message unique ID |
| action_unique_id | string | Yes | The action unique ID |

**cURL Example:**

```bash
curl -s -X GET "$BLOCKS_API_URL/conversations/conv_def456/messages/msg_001/actions/act_001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "act_001",
    "type": "message_actions",
    "attributes": {
      "unique_id": "act_001",
      "action_type": "button",
      "control_type": "submit",
      "control_label": "Approve",
      "control_value": "approved",
      "status": "pending",
      "enabled": "true",
      "payload": { "request_id": "req_001" },
      "response": null,
      "created_at": "2025-01-15T10:00:00Z",
      "updated_at": "2025-01-15T10:00:00Z"
    },
    "relationships": {
      "message": {
        "data": { "id": "msg_001", "type": "messages" }
      }
    }
  }
}
```

---

### Update Action

Update an action's fields, status, payload, or response. Commonly used to record a user's response to an action (e.g., clicking a button).

**Endpoint:** `PUT /conversations/:unique_id/messages/:message_unique_id/actions/:action_unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| message_unique_id | string | Yes | The message unique ID |
| action_unique_id | string | Yes | The action unique ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| action_type | string | No | Updated action type |
| control_type | string | No | Updated control type |
| control_label | string | No | Updated label |
| control_value | string | No | Updated value |
| control_name | string | No | Updated name |
| control_id | string | No | Updated id |
| control_class | string | No | Updated CSS class |
| control_tag | string | No | Updated tag |
| control_placeholder | string | No | Updated placeholder |
| control_href | string | No | Updated href |
| control_src | string | No | Updated src |
| control_alt | string | No | Updated alt |
| control_help | string | No | Updated help text |
| control_set_id | string | No | Updated set ID |
| control_set_name | string | No | Updated set name |
| control_set_class | string | No | Updated set class |
| payload | object | No | Updated JSONB payload |
| response | object | No | Updated JSONB response |
| status | string | No | Updated status |
| user_unique_id | string | No | User who interacted |

**cURL Example:**

```bash
curl -s -X PUT "$BLOCKS_API_URL/conversations/conv_def456/messages/msg_001/actions/act_001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message_action": {
      "status": "completed",
      "user_unique_id": "usr_xyz789",
      "response": {
        "choice": "approved",
        "comment": "Looks good to me",
        "responded_at": "2025-01-15T11:00:00Z"
      }
    }
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "act_001",
    "type": "message_actions",
    "attributes": {
      "unique_id": "act_001",
      "action_type": "button",
      "control_type": "submit",
      "control_label": "Approve",
      "control_value": "approved",
      "status": "completed",
      "user_unique_id": "usr_xyz789",
      "payload": { "request_id": "req_001" },
      "response": {
        "choice": "approved",
        "comment": "Looks good to me",
        "responded_at": "2025-01-15T11:00:00Z"
      },
      "updated_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

### Delete Action

Soft-delete an action by setting `enabled: 'false'` and `status: 'deleted'`. Returns 204 No Content.

**Endpoint:** `DELETE /conversations/:unique_id/messages/:message_unique_id/actions/:action_unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |
| message_unique_id | string | Yes | The message unique ID |
| action_unique_id | string | Yes | The action unique ID |

**cURL Example:**

```bash
curl -s -X DELETE "$BLOCKS_API_URL/conversations/conv_def456/messages/msg_001/actions/act_002" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response:** `204 No Content`
