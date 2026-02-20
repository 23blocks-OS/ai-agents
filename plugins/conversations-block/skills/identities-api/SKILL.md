---
name: conversations-identities-api
description: Manage user identities, registration, status, and WebSocket tokens in the 23blocks Conversations Block
version: 1.0.0
tags:
  - conversations
  - identities
  - users
  - status
  - websocket
env:
  - BLOCKS_API_URL
  - BLOCKS_AUTH_TOKEN
  - BLOCKS_API_KEY
---

# Conversations Identities API

Manage user identities within the Conversations Block. Users must register their identity before accessing private endpoints. This skill also handles user status management and WebSocket token generation for real-time connectivity.

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

### List Users

Retrieve a list of registered users.

**Endpoint:** `GET /users/`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number for pagination |
| per_page | integer | No | Number of results per page (default: 25) |
| search | string | No | Search term to filter users |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/?page=1&per_page=25" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "usr_abc123",
      "type": "users",
      "attributes": {
        "unique_id": "usr_abc123",
        "email": "user@example.com",
        "display_name": "John Doe",
        "avatar_url": "https://cdn.23blocks.com/avatars/usr_abc123.png",
        "status": "online",
        "status_message": null,
        "created_at": "2025-01-15T10:30:00Z",
        "updated_at": "2025-01-15T10:30:00Z"
      }
    }
  ],
  "meta": {
    "total": 50,
    "page": 1,
    "per_page": 25
  }
}
```

---

### Get User

Retrieve details of a specific user.

**Endpoint:** `GET /users/:unique_id/`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the user |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "usr_abc123",
    "type": "users",
    "attributes": {
      "unique_id": "usr_abc123",
      "email": "user@example.com",
      "display_name": "John Doe",
      "avatar_url": "https://cdn.23blocks.com/avatars/usr_abc123.png",
      "status": "online",
      "status_message": "Available",
      "last_seen_at": "2025-01-15T10:30:00Z",
      "created_at": "2025-01-15T10:30:00Z",
      "updated_at": "2025-01-15T10:30:00Z"
    }
  }
}
```

---

### Register User

Register a user identity within the Conversations Block. This is required before the user can access private endpoints.

**Endpoint:** `POST /users/:unique_id/register/`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the user to register |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| display_name | string | No | Display name for the user |
| email | string | No | Email address |
| avatar_url | string | No | URL to user avatar |
| metadata | object | No | Additional metadata |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/users/usr_abc123/register/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "display_name": "John Doe",
    "email": "user@example.com",
    "avatar_url": "https://example.com/avatar.png",
    "metadata": {
      "role": "member"
    }
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "usr_abc123",
    "type": "users",
    "attributes": {
      "unique_id": "usr_abc123",
      "email": "user@example.com",
      "display_name": "John Doe",
      "avatar_url": "https://example.com/avatar.png",
      "status": "offline",
      "status_message": null,
      "metadata": {
        "role": "member"
      },
      "created_at": "2025-01-15T10:30:00Z",
      "updated_at": "2025-01-15T10:30:00Z"
    }
  }
}
```

---

### Update User

Update a registered user's profile information.

**Endpoint:** `PUT /users/:unique_id/`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the user |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| display_name | string | No | Updated display name |
| email | string | No | Updated email address |
| avatar_url | string | No | Updated avatar URL |
| metadata | object | No | Updated metadata |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/users/usr_abc123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "display_name": "John D.",
    "metadata": {
      "role": "admin"
    }
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "usr_abc123",
    "type": "users",
    "attributes": {
      "unique_id": "usr_abc123",
      "email": "user@example.com",
      "display_name": "John D.",
      "avatar_url": "https://example.com/avatar.png",
      "status": "online",
      "status_message": null,
      "metadata": {
        "role": "admin"
      },
      "created_at": "2025-01-15T10:30:00Z",
      "updated_at": "2025-01-16T08:00:00Z"
    }
  }
}
```

---

### Get User Status

Retrieve the current status of a user.

**Endpoint:** `GET /users/:unique_id/status`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The unique identifier of the user |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/status" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "usr_abc123",
    "type": "user_status",
    "attributes": {
      "unique_id": "usr_abc123",
      "status": "online",
      "status_message": "In a meeting",
      "status_emoji": ":calendar:",
      "expires_at": "2025-01-15T12:00:00Z",
      "last_seen_at": "2025-01-15T10:30:00Z"
    }
  }
}
```

---

### Set User Status

Set the current user's status message and availability.

**Endpoint:** `POST /users/status`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| status | string | Yes | Status value: `online`, `away`, `busy`, `offline` |
| status_message | string | No | Custom status message |
| status_emoji | string | No | Emoji code for status |
| expires_at | string | No | ISO 8601 expiration time for the status |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/users/status" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "busy",
    "status_message": "In a meeting",
    "status_emoji": ":calendar:",
    "expires_at": "2025-01-15T12:00:00Z"
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "usr_abc123",
    "type": "user_status",
    "attributes": {
      "status": "busy",
      "status_message": "In a meeting",
      "status_emoji": ":calendar:",
      "expires_at": "2025-01-15T12:00:00Z"
    }
  }
}
```

---

### Clear User Status

Clear the current user's custom status.

**Endpoint:** `DELETE /users/status`

**cURL Example:**

```bash
curl -s -X DELETE "https://conversations.api.us.23blocks.com/users/status" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "usr_abc123",
    "type": "user_status",
    "attributes": {
      "status": "online",
      "status_message": null,
      "status_emoji": null,
      "expires_at": null
    }
  }
}
```

---

### Generate WebSocket Token

Generate a token for establishing a WebSocket connection for real-time messaging.

**Endpoint:** `POST /ws-tokens`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| user_unique_id | string | Yes | Unique ID of the user requesting the token |
| device_id | string | No | Device identifier for multi-device support |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/ws-tokens" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "usr_abc123",
    "device_id": "device_001"
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "wst_xyz789",
    "type": "ws_tokens",
    "attributes": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "ws_url": "wss://conversations.ws.us.23blocks.com",
      "user_unique_id": "usr_abc123",
      "device_id": "device_001",
      "expires_at": "2025-01-15T11:30:00Z",
      "created_at": "2025-01-15T10:30:00Z"
    }
  }
}
```

---

## Data Model

### User

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the user |
| email | string | User email address |
| display_name | string | User display name |
| avatar_url | string | URL to user avatar image |
| status | string | Current status: `online`, `away`, `busy`, `offline` |
| status_message | string | Custom status message |
| status_emoji | string | Emoji code for status display |
| metadata | object | Arbitrary key-value metadata |
| last_seen_at | datetime | Timestamp of last activity |
| created_at | datetime | Account creation timestamp |
| updated_at | datetime | Last update timestamp |

### WebSocket Token

| Field | Type | Description |
|-------|------|-------------|
| token | string | JWT token for WebSocket authentication |
| ws_url | string | WebSocket server URL |
| user_unique_id | string | Associated user ID |
| device_id | string | Device identifier |
| expires_at | datetime | Token expiration time |
| created_at | datetime | Token creation timestamp |

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
      "detail": "User with unique_id 'usr_invalid' not found"
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
      "detail": "User is already registered in this block"
    }
  ]
}
```
