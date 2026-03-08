# CRM Zoom API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Zoom Meeting Endpoints

### GET /users/:user_unique_id/meetings/:meeting_unique_id/zoom - Get Zoom Meeting

Retrieves Zoom meeting details linked to a CRM meeting.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/meetings/meeting-uuid-456/zoom" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "zoom-uuid-123",
    "type": "zoom_meeting",
    "attributes": {
      "unique_id": "zoom-uuid-123",
      "meeting_id": "meeting-uuid-456",
      "zoom_meeting_id": "85012345678",
      "join_url": "https://zoom.us/j/85012345678?pwd=abc123",
      "host_email": "host@example.com",
      "duration": 60,
      "status": "waiting",
      "created_at": "2026-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Zoom meeting not found

---

### POST /users/:user_unique_id/meetings/:meeting_unique_id/zoom - Create Zoom Meeting

Creates a Zoom meeting linked to a CRM meeting.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/meetings/meeting-uuid-456/zoom" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "zoom": {
      "duration": 60,
      "host_email": "host@example.com"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `duration` | integer | No | Meeting duration in minutes |
| `host_email` | string | No | Zoom host email |

**Response 201:**
```json
{
  "data": {
    "id": "zoom-uuid-456",
    "type": "zoom_meeting",
    "attributes": {
      "unique_id": "zoom-uuid-456",
      "meeting_id": "meeting-uuid-456",
      "zoom_meeting_id": "85098765432",
      "join_url": "https://zoom.us/j/85098765432?pwd=xyz789",
      "host_email": "host@example.com",
      "duration": 60,
      "status": "waiting",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /users/:user_unique_id/meetings/:meeting_unique_id/zoom - Update Zoom Meeting

Updates an existing Zoom meeting.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/meetings/meeting-uuid-456/zoom" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "zoom": {
      "duration": 90
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "zoom-uuid-123",
    "type": "zoom_meeting",
    "attributes": {
      "unique_id": "zoom-uuid-123",
      "duration": 90,
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /users/:user_unique_id/meetings/:meeting_unique_id/zoom - Delete Zoom Meeting

Deletes a Zoom meeting linked to a CRM meeting.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/user-uuid-123/meetings/meeting-uuid-456/zoom" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### GET /users/:user_unique_id/zoom/availability - User Zoom Availability

Checks a user's Zoom availability.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/zoom/availability?date=2026-03-15" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `date` | date | No | Date to check availability for |

**Response 200:**
```json
{
  "data": {
    "user_id": "user-uuid-123",
    "date": "2026-03-15",
    "available": true,
    "busy_slots": [
      {
        "start_time": "2026-03-15T10:00:00Z",
        "end_time": "2026-03-15T11:00:00Z"
      }
    ],
    "available_slots": [
      {
        "start_time": "2026-03-15T09:00:00Z",
        "end_time": "2026-03-15T10:00:00Z"
      },
      {
        "start_time": "2026-03-15T11:00:00Z",
        "end_time": "2026-03-15T17:00:00Z"
      }
    ]
  }
}
```

---

### POST /zoom/webhooks - Zoom Webhook

Handles incoming Zoom webhook events.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/zoom/webhooks" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "event": "meeting.ended",
    "payload": {
      "meeting_id": "85012345678",
      "duration": 55
    }
  }'
```

**Response 200:**
```json
{
  "message": "Webhook processed successfully"
}
```

---

## Zoom Host Endpoints

### GET /zoom_hosts/available_users - Available Zoom Users

Lists users available as Zoom hosts.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/zoom_hosts/available_users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "user-uuid-123",
      "type": "user",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "host@example.com",
        "name": "John Doe",
        "zoom_licensed": true
      }
    }
  ]
}
```

---

### GET /zoom_hosts/ - List Zoom Hosts

Lists all configured Zoom hosts.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/zoom_hosts/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "zoom-host-uuid-123",
      "type": "zoom_host",
      "attributes": {
        "unique_id": "zoom-host-uuid-123",
        "user_id": "user-uuid-123",
        "email": "host@example.com",
        "name": "John Doe",
        "max_concurrent_meetings": 3,
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /zoom_hosts/:unique_id - Get Zoom Host

Retrieves a specific Zoom host.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/zoom_hosts/zoom-host-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "zoom-host-uuid-123",
    "type": "zoom_host",
    "attributes": {
      "unique_id": "zoom-host-uuid-123",
      "user_id": "user-uuid-123",
      "email": "host@example.com",
      "name": "John Doe",
      "max_concurrent_meetings": 3,
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Zoom host not found

---

### POST /zoom_hosts/ - Create Zoom Host

Configures a new Zoom host.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/zoom_hosts/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "zoom_host": {
      "user_id": "user-uuid-456",
      "email": "newhost@example.com",
      "max_concurrent_meetings": 5
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | uuid | Yes | Associated user ID |
| `email` | string | Yes | Zoom host email |
| `max_concurrent_meetings` | integer | No | Max concurrent meetings (default: 3) |

**Response 201:**
```json
{
  "data": {
    "id": "zoom-host-uuid-456",
    "type": "zoom_host",
    "attributes": {
      "unique_id": "zoom-host-uuid-456",
      "user_id": "user-uuid-456",
      "email": "newhost@example.com",
      "max_concurrent_meetings": 5,
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /zoom_hosts/:unique_id - Update Zoom Host

Updates an existing Zoom host configuration.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/zoom_hosts/zoom-host-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "zoom_host": {
      "max_concurrent_meetings": 5
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "zoom-host-uuid-123",
    "type": "zoom_host",
    "attributes": {
      "unique_id": "zoom-host-uuid-123",
      "max_concurrent_meetings": 5,
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /zoom_hosts/:unique_id - Delete Zoom Host

Deletes a Zoom host configuration.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/zoom_hosts/zoom-host-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### GET /zoom_hosts/:unique_id/availability - Host Availability

Checks a Zoom host's availability.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/zoom_hosts/zoom-host-uuid-123/availability?date=2026-03-15" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "host_id": "zoom-host-uuid-123",
    "date": "2026-03-15",
    "current_meetings": 1,
    "max_concurrent_meetings": 3,
    "available": true,
    "busy_slots": [
      {
        "start_time": "2026-03-15T10:00:00Z",
        "end_time": "2026-03-15T11:00:00Z",
        "meeting_id": "meeting-uuid-789"
      }
    ]
  }
}
```

---

### GET /zoom_hosts/:unique_id/allocations - Host Allocations

Retrieves meeting allocations for a Zoom host.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/zoom_hosts/zoom-host-uuid-123/allocations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "meeting_id": "meeting-uuid-789",
      "zoom_meeting_id": "85012345678",
      "start_time": "2026-03-15T10:00:00Z",
      "end_time": "2026-03-15T11:00:00Z",
      "status": "scheduled"
    }
  ],
  "meta": {
    "total_allocations": 5
  }
}
```
