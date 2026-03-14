---
name: 23blocks-conversations-message-actions-api
description: Manage interactive message actions (buttons, links, controls) within conversation messages. Use when listing, updating, or deleting actions on messages, or creating actions inline during message creation.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Message Actions API

Manage interactive actions attached to conversation messages. Actions represent UI controls (buttons, links, inputs) that can be embedded in messages. Actions are created inline when sending a message and can be listed, updated, or deleted afterwards.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Conversations API base URL | `https://conversations.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

```bash
curl -X GET "$BLOCKS_API_URL/conversations/{unique_id}/messages/{message_unique_id}/actions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/conversations/:unique_id/messages/:message_unique_id/actions` | List actions on a message |
| GET | `/conversations/:unique_id/messages/:message_unique_id/actions/:action_unique_id` | Get a specific action |
| PUT | `/conversations/:unique_id/messages/:message_unique_id/actions/:action_unique_id` | Update an action |
| DELETE | `/conversations/:unique_id/messages/:message_unique_id/actions/:action_unique_id` | Delete an action |

> **Note:** Actions are created inline during message creation via `POST /conversations/:unique_id/messages` by passing an `actions` array in the message body. See the **23blocks-conversations-messages-api** skill for details.

---

## Data Model

### MessageAction

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| unique_id | string | auto | Unique identifier for the action |
| action_type | string | Yes | Action type (max 50 chars), e.g. `button`, `link`, `input` |
| control_type | string | Yes | Control type (max 50 chars), e.g. `submit`, `cancel`, `navigate` |
| control_label | string | Yes | Display label (max 500 chars) |
| control_value | string | Yes | Control value (max 2000 chars) |
| control_name | string | No | Control name attribute (max 255 chars) |
| control_id | string | No | Control id attribute (max 255 chars) |
| control_class | string | No | CSS class attribute (max 255 chars) |
| control_tag | string | No | HTML tag override (max 100 chars) |
| control_placeholder | string | No | Placeholder text (max 500 chars) |
| control_href | string | No | Link URL (max 2000 chars) |
| control_src | string | No | Image/media source URL (max 2000 chars) |
| control_alt | string | No | Alt text (max 500 chars) |
| control_help | string | No | Help text (max 1000 chars) |
| control_set_id | string | No | Group actions into a set by ID |
| control_set_name | string | No | Set display name |
| control_set_class | string | No | Set CSS class |
| payload | object (JSONB) | No | Arbitrary JSON payload data |
| response | object (JSONB) | No | Arbitrary JSON response data |
| status | string | auto | Status: `pending`, `completed`, `cancelled` |
| enabled | string | auto | `true` or `false` (soft delete) |
| user_unique_id | string | No | User who interacted with the action |
| created_at | datetime | auto | Creation timestamp |
| updated_at | datetime | auto | Last update timestamp |

### Scopes

- **active** — only enabled actions (`enabled = 'true'`)
- **pending** — actions with `status = 'pending'`
- **by_status(status)** — filter by status
- **by_control_set(set_id)** — filter by control_set_id

---

## WebSocket Events

Actions broadcast to `conversation_{context_id}` channel on create/update with `object_type: 'message_action'`.

---

## Error Response Format

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

Common status codes: `401` Unauthorized, `403` Forbidden, `404` Not Found, `422` Unprocessable Entity.
