---
name: 23blocks-conversations-notifications-api
description: "Conversations Block notifications: settings, do-not-disturb, send, web push, VAPID keys. Use for notifying users."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Notifications API

Manage notification settings, create and send notifications, web push notifications, and VAPID key setup for push notification delivery.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://realtime.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users/:unique_id/notifications/settings` | Get notification settings |
| PUT | `/users/:unique_id/notifications/settings` | Update notification settings |
| GET | `/users/:unique_id/notifications/` | List user notifications |
| POST | `/notifications/` | Create notification |
| GET | `/notifications/:unique_id` | Get notification |
| PUT | `/notifications/:unique_id` | Update notification |
| PUT | `/notifications/` | Bulk update notifications |
| POST | `/web_notifications/` | Send web notification |
| POST | `/notify` | Send push notification |
| POST | `/companies/:unique_id/vapid` | Setup VAPID keys |
| GET | `/entities/:unique_id/notifications` | Get entity notifications |
| GET | `/sources/:unique_id` | Get notification source |

---

## Data Model

### Notification

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the notification |
| user_id | string | Target user unique ID |
| title | string | Notification title |
| body | string | Notification body text |
| notification_type | string | Type: `message`, `mention`, `invite`, `system`, `custom` |
| read | boolean | Whether the notification has been read |
| read_at | datetime | Timestamp when marked as read |
| action_url | string | URL to navigate to on click |
| status | string | Delivery status: `pending`, `delivered`, `failed` |
| source_id | string | ID of the originating object |
| source_type | string | Type of the originating object |
| event_name | string | Business event name (e.g. `pcu_test_created`) used to select the notification template — the mailer falls back to `source_type` when blank |
| metadata | object | Arbitrary key-value metadata |
| created_at | datetime | Notification creation timestamp |

### Notification Settings

| Field | Type | Description |
|-------|------|-------------|
| user_unique_id | string | Associated user ID |
| push_enabled | boolean | Push notifications enabled |
| email_enabled | boolean | Email notifications enabled |
| web_enabled | boolean | Web notifications enabled |
| sound_enabled | boolean | Notification sounds enabled |
| do_not_disturb | boolean | DND mode enabled |
| dnd_start | string | DND start time (HH:MM) |
| dnd_end | string | DND end time (HH:MM) |
| muted_conversations | array | List of muted conversation IDs |
| muted_groups | array | List of muted group IDs |

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

Common status codes: `401` Unauthorized, `404` Not Found, `422` Unprocessable Entity.

---

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-conversations`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useConversationsBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// NotificationsService — client.conversations.notifications
list(params?: ListNotificationsParams): Promise<PageResult<Notification>>;
get(uniqueId: string): Promise<Notification>;
create(data: CreateNotificationRequest): Promise<Notification>;
update(uniqueId: string, data: UpdateNotificationRequest): Promise<Notification>;
delete(uniqueId: string): Promise<void>;
markAsRead(uniqueId: string): Promise<Notification>;
markAsUnread(uniqueId: string): Promise<Notification>;
listByTarget(targetId: string, params?: ListNotificationsParams): Promise<PageResult<Notification>>;
listUnread(params?: ListNotificationsParams): Promise<PageResult<Notification>>;
```

### TypeScript Types

```typescript
import type {
  Notification,
  CreateNotificationRequest,
  UpdateNotificationRequest,
  ListNotificationsParams,
} from '@23blocks/block-conversations';
```
