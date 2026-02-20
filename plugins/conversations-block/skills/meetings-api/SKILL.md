---
name: conversations-meetings-api
description: Create and manage video/audio meetings and Vonage sessions in the 23blocks Conversations Block
version: 1.0.0
tags:
  - conversations
  - meetings
  - video
  - vonage
  - real-time
env:
  - BLOCKS_API_URL
  - BLOCKS_AUTH_TOKEN
  - BLOCKS_API_KEY
---

# Meetings API

Create and manage video/audio meetings within conversations. Supports Vonage session creation for real-time audio and video communication.

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

### Create Meeting

Create a new meeting associated with a conversation.

**Endpoint:** `POST /meetings/:unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID to create the meeting in |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| title | string | No | Meeting title |
| provider | string | No | Meeting provider: `vonage` (default), `custom` |
| scheduled_start | string | No | ISO 8601 scheduled start time |
| scheduled_end | string | No | ISO 8601 scheduled end time |
| participants | array | No | Array of user unique IDs to invite |
| settings | object | No | Meeting settings (video, audio, recording) |
| metadata | object | No | Additional metadata |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/meetings/conv_def456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Sprint Planning",
    "provider": "vonage",
    "scheduled_start": "2025-01-16T14:00:00Z",
    "scheduled_end": "2025-01-16T15:00:00Z",
    "participants": ["usr_abc123", "usr_xyz789", "usr_lmn456"],
    "settings": {
      "video_enabled": true,
      "audio_enabled": true,
      "recording_enabled": false,
      "screen_sharing_enabled": true
    },
    "metadata": {
      "sprint": "Sprint 5",
      "recurring": false
    }
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "meet_001",
    "type": "meetings",
    "attributes": {
      "unique_id": "meet_001",
      "conversation_id": "conv_def456",
      "title": "Sprint Planning",
      "provider": "vonage",
      "session_id": "2_MX40NTYzMTI3Mn5-MTU0NzA4MDQ...",
      "join_url": "https://meetings.23blocks.com/meet_001",
      "scheduled_start": "2025-01-16T14:00:00Z",
      "scheduled_end": "2025-01-16T15:00:00Z",
      "start_time": null,
      "end_time": null,
      "participants": ["usr_abc123", "usr_xyz789", "usr_lmn456"],
      "active_participants": [],
      "settings": {
        "video_enabled": true,
        "audio_enabled": true,
        "recording_enabled": false,
        "screen_sharing_enabled": true
      },
      "status": "scheduled",
      "metadata": {
        "sprint": "Sprint 5",
        "recurring": false
      },
      "created_at": "2025-01-15T11:00:00Z",
      "updated_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

### Create Vonage Session

Create a Vonage video session for a conversation, returning the session credentials needed for clients to connect.

**Endpoint:** `POST /vonage/:unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The conversation unique ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| user_unique_id | string | Yes | Unique ID of the user joining the session |
| role | string | No | User role: `publisher` (default), `subscriber`, `moderator` |
| session_type | string | No | Session type: `relayed` (default), `routed` |
| expire_time | integer | No | Token expiration time in seconds (default: 3600) |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/vonage/conv_def456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "usr_abc123",
    "role": "publisher",
    "session_type": "routed",
    "expire_time": 7200
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "vs_001",
    "type": "vonage_sessions",
    "attributes": {
      "session_id": "2_MX40NTYzMTI3Mn5-MTU0NzA4MDQ...",
      "token": "T1==cGFydG5lcl9pZD0xMjM0NTY3OCZzaWc9...",
      "api_key": "12345678",
      "conversation_id": "conv_def456",
      "user_unique_id": "usr_abc123",
      "role": "publisher",
      "session_type": "routed",
      "expires_at": "2025-01-15T13:00:00Z",
      "created_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

## Data Model

### Meeting

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the meeting |
| conversation_id | string | Associated conversation ID |
| title | string | Meeting title |
| provider | string | Meeting provider: `vonage`, `custom` |
| session_id | string | Provider session identifier |
| join_url | string | URL for participants to join the meeting |
| scheduled_start | datetime | Scheduled start time |
| scheduled_end | datetime | Scheduled end time |
| start_time | datetime | Actual start time (null until started) |
| end_time | datetime | Actual end time (null until ended) |
| participants | array | List of invited participant user IDs |
| active_participants | array | List of currently connected participant user IDs |
| settings | object | Meeting settings configuration |
| status | string | Status: `scheduled`, `active`, `ended`, `cancelled` |
| metadata | object | Arbitrary key-value metadata |
| created_at | datetime | Meeting creation timestamp |
| updated_at | datetime | Last update timestamp |

### Meeting Settings

| Field | Type | Description |
|-------|------|-------------|
| video_enabled | boolean | Whether video is enabled |
| audio_enabled | boolean | Whether audio is enabled |
| recording_enabled | boolean | Whether recording is enabled |
| screen_sharing_enabled | boolean | Whether screen sharing is allowed |

### Vonage Session

| Field | Type | Description |
|-------|------|-------------|
| session_id | string | Vonage session identifier |
| token | string | Client authentication token |
| api_key | string | Vonage API key for client SDK initialization |
| conversation_id | string | Associated conversation ID |
| user_unique_id | string | User the token was generated for |
| role | string | User role: `publisher`, `subscriber`, `moderator` |
| session_type | string | Session type: `relayed`, `routed` |
| expires_at | datetime | Token expiration time |
| created_at | datetime | Session creation timestamp |

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
      "detail": "User is not a participant in this conversation"
    }
  ]
}
```

### 503 Service Unavailable

```json
{
  "errors": [
    {
      "status": "503",
      "title": "Service Unavailable",
      "detail": "Vonage service is temporarily unavailable. Please try again later."
    }
  ]
}
```
