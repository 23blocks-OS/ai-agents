---
name: conversations-notifications-api
description: Manage notification settings, create and send notifications, web push, and VAPID key setup in the 23blocks Conversations Block
version: 1.0.0
tags:
  - conversations
  - notifications
  - push
  - vapid
  - web-notifications
env:
  - BLOCKS_API_URL
  - BLOCKS_AUTH_TOKEN
  - BLOCKS_API_KEY
---

# Notifications API

Manage notification settings, create and send notifications, web push notifications, and VAPID key setup for push notification delivery.

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

## Notification Settings

### Get Notification Settings

Retrieve notification preferences for a user.

**Endpoint:** `GET /users/:unique_id/notifications/settings`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The user's unique identifier |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/notifications/settings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "ns_001",
    "type": "notification_settings",
    "attributes": {
      "user_unique_id": "usr_abc123",
      "push_enabled": true,
      "email_enabled": false,
      "web_enabled": true,
      "sound_enabled": true,
      "do_not_disturb": false,
      "dnd_start": null,
      "dnd_end": null,
      "muted_conversations": [],
      "muted_groups": [],
      "updated_at": "2025-01-15T10:00:00Z"
    }
  }
}
```

---

### Update Notification Settings

Update notification preferences for a user.

**Endpoint:** `PUT /users/:unique_id/notifications/settings`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The user's unique identifier |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| push_enabled | boolean | No | Enable/disable push notifications |
| email_enabled | boolean | No | Enable/disable email notifications |
| web_enabled | boolean | No | Enable/disable web notifications |
| sound_enabled | boolean | No | Enable/disable notification sounds |
| do_not_disturb | boolean | No | Enable/disable DND mode |
| dnd_start | string | No | DND start time (HH:MM format) |
| dnd_end | string | No | DND end time (HH:MM format) |
| muted_conversations | array | No | List of muted conversation IDs |
| muted_groups | array | No | List of muted group IDs |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/users/usr_abc123/notifications/settings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "push_enabled": true,
    "email_enabled": true,
    "do_not_disturb": true,
    "dnd_start": "22:00",
    "dnd_end": "08:00"
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "ns_001",
    "type": "notification_settings",
    "attributes": {
      "user_unique_id": "usr_abc123",
      "push_enabled": true,
      "email_enabled": true,
      "web_enabled": true,
      "sound_enabled": true,
      "do_not_disturb": true,
      "dnd_start": "22:00",
      "dnd_end": "08:00",
      "muted_conversations": [],
      "muted_groups": [],
      "updated_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

## Notifications

### List User Notifications

Retrieve all notifications for a user.

**Endpoint:** `GET /users/:unique_id/notifications/`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The user's unique identifier |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number for pagination |
| per_page | integer | No | Results per page (default: 25) |
| read | boolean | No | Filter by read status |
| notification_type | string | No | Filter by type |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/notifications/?page=1&per_page=25&read=false" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "notif_001",
      "type": "notifications",
      "attributes": {
        "unique_id": "notif_001",
        "user_id": "usr_abc123",
        "title": "New message from Jane",
        "body": "Hey, are you available for a quick call?",
        "notification_type": "message",
        "read": false,
        "action_url": "/conversations/conv_def456",
        "status": "delivered",
        "source_id": "msg_001",
        "source_type": "message",
        "created_at": "2025-01-15T10:30:00Z"
      }
    }
  ],
  "meta": {
    "total": 8,
    "unread_count": 5,
    "page": 1,
    "per_page": 25
  }
}
```

---

### Create Notification

Create a new notification for a user.

**Endpoint:** `POST /notifications/`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| user_id | string | Yes | Target user unique ID |
| title | string | Yes | Notification title |
| body | string | Yes | Notification body text |
| notification_type | string | No | Type: `message`, `mention`, `invite`, `system`, `custom` |
| action_url | string | No | URL to navigate to on click |
| source_id | string | No | ID of the source object |
| source_type | string | No | Type of the source object |
| metadata | object | No | Additional metadata |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/notifications/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "usr_abc123",
    "title": "You were mentioned",
    "body": "Jane mentioned you in Engineering Team",
    "notification_type": "mention",
    "action_url": "/conversations/conv_grp789",
    "source_id": "msg_mention001",
    "source_type": "message"
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "notif_new001",
    "type": "notifications",
    "attributes": {
      "unique_id": "notif_new001",
      "user_id": "usr_abc123",
      "title": "You were mentioned",
      "body": "Jane mentioned you in Engineering Team",
      "notification_type": "mention",
      "read": false,
      "action_url": "/conversations/conv_grp789",
      "status": "pending",
      "source_id": "msg_mention001",
      "source_type": "message",
      "created_at": "2025-01-15T11:30:00Z"
    }
  }
}
```

---

### Get Notification

Retrieve a specific notification.

**Endpoint:** `GET /notifications/:unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The notification's unique identifier |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/notifications/notif_001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "notif_001",
    "type": "notifications",
    "attributes": {
      "unique_id": "notif_001",
      "user_id": "usr_abc123",
      "title": "New message from Jane",
      "body": "Hey, are you available for a quick call?",
      "notification_type": "message",
      "read": false,
      "action_url": "/conversations/conv_def456",
      "status": "delivered",
      "source_id": "msg_001",
      "source_type": "message",
      "metadata": {},
      "created_at": "2025-01-15T10:30:00Z"
    }
  }
}
```

---

### Update Notification

Update a notification (e.g., mark as read).

**Endpoint:** `PUT /notifications/:unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The notification's unique identifier |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| read | boolean | No | Mark as read/unread |
| status | string | No | Update status |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/notifications/notif_001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "read": true
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "notif_001",
    "type": "notifications",
    "attributes": {
      "unique_id": "notif_001",
      "read": true,
      "read_at": "2025-01-15T11:00:00Z",
      "updated_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

### Bulk Update Notifications

Update multiple notifications at once (e.g., mark all as read).

**Endpoint:** `PUT /notifications/`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| user_id | string | Yes | The user whose notifications to update |
| read | boolean | No | Mark all as read/unread |
| notification_ids | array | No | Specific notification IDs to update (omit for all) |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/notifications/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "usr_abc123",
    "read": true,
    "notification_ids": ["notif_001", "notif_002", "notif_003"]
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "type": "bulk_update",
    "attributes": {
      "updated_count": 3,
      "user_id": "usr_abc123",
      "updated_at": "2025-01-15T12:00:00Z"
    }
  }
}
```

---

### Send Web Notification

Send a web browser notification to a user.

**Endpoint:** `POST /web_notifications/`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| user_id | string | Yes | Target user unique ID |
| title | string | Yes | Notification title |
| body | string | Yes | Notification body text |
| icon | string | No | URL to notification icon |
| action_url | string | No | URL to open on click |
| tag | string | No | Tag for notification grouping |
| require_interaction | boolean | No | Keep notification until user interacts |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/web_notifications/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "usr_abc123",
    "title": "New message",
    "body": "You have a new message from Jane",
    "icon": "https://cdn.23blocks.com/icons/message.png",
    "action_url": "/conversations/conv_def456",
    "tag": "message_conv_def456"
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "webnotif_001",
    "type": "web_notifications",
    "attributes": {
      "user_id": "usr_abc123",
      "title": "New message",
      "body": "You have a new message from Jane",
      "delivered": true,
      "delivered_at": "2025-01-15T12:30:00Z"
    }
  }
}
```

---

### Send Push Notification

Send a push notification to a user's device.

**Endpoint:** `POST /notify`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| user_id | string | Yes | Target user unique ID |
| title | string | Yes | Push notification title |
| body | string | Yes | Push notification body |
| data | object | No | Custom data payload |
| priority | string | No | Priority: `normal` (default), `high` |
| ttl | integer | No | Time-to-live in seconds |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/notify" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "usr_abc123",
    "title": "Meeting starting",
    "body": "Your meeting with the Engineering Team starts in 5 minutes",
    "data": {
      "conversation_id": "conv_grp789",
      "meeting_id": "meet_001"
    },
    "priority": "high",
    "ttl": 300
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "push_001",
    "type": "push_notifications",
    "attributes": {
      "user_id": "usr_abc123",
      "title": "Meeting starting",
      "body": "Your meeting with the Engineering Team starts in 5 minutes",
      "delivered": true,
      "device_count": 2,
      "created_at": "2025-01-15T13:00:00Z"
    }
  }
}
```

---

### Setup VAPID Keys

Configure VAPID keys for web push notification delivery for a company.

**Endpoint:** `POST /companies/:unique_id/vapid`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The company's unique identifier |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| subject | string | Yes | VAPID subject (mailto: URL or website URL) |
| public_key | string | No | VAPID public key (auto-generated if omitted) |
| private_key | string | No | VAPID private key (auto-generated if omitted) |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/companies/comp_001/vapid" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "mailto:admin@example.com"
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "vapid_001",
    "type": "vapid_keys",
    "attributes": {
      "company_unique_id": "comp_001",
      "subject": "mailto:admin@example.com",
      "public_key": "BNbxGYNMhEIi9zrneh7mqh4gHhTEvRhJBRaFYgO...",
      "configured": true,
      "created_at": "2025-01-15T14:00:00Z"
    }
  }
}
```

---

### Get Entity Notifications

Retrieve notifications associated with a specific entity (conversation, group, etc.).

**Endpoint:** `GET /entities/:unique_id/notifications`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The entity's unique identifier |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number for pagination |
| per_page | integer | No | Results per page (default: 25) |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/entities/conv_def456/notifications?page=1&per_page=25" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "notif_001",
      "type": "notifications",
      "attributes": {
        "unique_id": "notif_001",
        "user_id": "usr_abc123",
        "title": "New message from Jane",
        "body": "Hey, are you available for a quick call?",
        "notification_type": "message",
        "source_id": "conv_def456",
        "source_type": "conversation",
        "created_at": "2025-01-15T10:30:00Z"
      }
    }
  ],
  "meta": {
    "total": 3,
    "page": 1,
    "per_page": 25
  }
}
```

---

### Get Notification Source

Retrieve details about a notification source.

**Endpoint:** `GET /sources/:unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The source's unique identifier |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/sources/src_001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "src_001",
    "type": "notification_sources",
    "attributes": {
      "unique_id": "src_001",
      "source_type": "conversation",
      "source_id": "conv_def456",
      "name": "Direct Message - John & Jane",
      "description": "Notifications from direct message conversation",
      "notification_count": 15,
      "created_at": "2025-01-10T08:00:00Z"
    }
  }
}
```

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
      "detail": "Notification with unique_id 'notif_invalid' not found"
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
      "detail": "VAPID keys are already configured for this company"
    }
  ]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-conversations
```

### Setup

```typescript
import { create23BlocksClient } from '@23blocks/sdk';

const client = create23BlocksClient({
  authToken: process.env.BLOCKS_AUTH_TOKEN!,
  apiKey: process.env.BLOCKS_API_KEY!,
  apiUrl: process.env.BLOCKS_API_URL!,
});
```

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

### React Hook

```typescript
import { useConversationsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useConversationsBlock();
  const result = await client.conversations.notifications.list();
}
```
