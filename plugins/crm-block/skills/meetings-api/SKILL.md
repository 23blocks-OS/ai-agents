---
name: crm-meetings-api
description: Manage 23blocks CRM meetings via REST API. Use when scheduling meetings, managing participants, and tracking meeting details.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# CRM Meetings API

Complete API reference for 23blocks CRM meeting management with participant handling.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://crm.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
echo "All credentials configured"
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | CRM API base URL | `https://crm.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/meetings/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /meetings/ - List Meetings

Lists all meetings with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/meetings/?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "meeting-uuid-123",
      "type": "meeting",
      "attributes": {
        "unique_id": "meeting-uuid-123",
        "title": "Quarterly Review",
        "description": "Q1 business review with stakeholders",
        "start_time": "2026-03-15T14:00:00Z",
        "end_time": "2026-03-15T15:00:00Z",
        "location": "Conference Room A",
        "meeting_type": "in_person",
        "status": "scheduled",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 4,
    "totalRecords": 56
  }
}
```

---

### GET /meetings/:unique_id/ - Get Meeting

Retrieves a single meeting by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/meetings/meeting-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "meeting-uuid-123",
    "type": "meeting",
    "attributes": {
      "unique_id": "meeting-uuid-123",
      "title": "Quarterly Review",
      "description": "Q1 business review with stakeholders",
      "start_time": "2026-03-15T14:00:00Z",
      "end_time": "2026-03-15T15:00:00Z",
      "location": "Conference Room A",
      "meeting_type": "in_person",
      "status": "scheduled",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    },
    "relationships": {
      "participants": {
        "data": [
          { "id": "participant-uuid", "type": "participant" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Meeting not found

---

### GET /meetings/:unique_id/participants - Meeting Participants

Lists all participants for a meeting.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/meetings/meeting-uuid-123/participants" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "participant-uuid-123",
      "type": "participant",
      "attributes": {
        "unique_id": "participant-uuid-123",
        "name": "Jane Smith",
        "email": "jane@acme.com",
        "role": "attendee",
        "rsvp_status": "accepted",
        "created_at": "2026-01-15T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /meetings/ - Create Meeting

Creates a new meeting.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/meetings/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "meeting": {
      "title": "Product Demo",
      "description": "Demo of new features for Acme Corp",
      "start_time": "2026-03-20T10:00:00Z",
      "end_time": "2026-03-20T11:00:00Z",
      "location": "Zoom",
      "meeting_type": "virtual"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | Meeting title |
| `description` | string | No | Meeting description |
| `start_time` | timestamp | Yes | Start time (ISO 8601) |
| `end_time` | timestamp | Yes | End time (ISO 8601) |
| `location` | string | No | Meeting location |
| `meeting_type` | string | No | in_person, virtual, phone |

**Response 201:**
```json
{
  "data": {
    "id": "meeting-uuid-456",
    "type": "meeting",
    "attributes": {
      "unique_id": "meeting-uuid-456",
      "title": "Product Demo",
      "description": "Demo of new features for Acme Corp",
      "start_time": "2026-03-20T10:00:00Z",
      "end_time": "2026-03-20T11:00:00Z",
      "location": "Zoom",
      "meeting_type": "virtual",
      "status": "scheduled",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### POST /meetings/:unique_id/participants - Add Participant

Adds a participant to a meeting.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/meetings/meeting-uuid-123/participants" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "participant": {
      "name": "Bob Wilson",
      "email": "bob@acme.com",
      "role": "attendee"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Participant name |
| `email` | string | Yes | Participant email |
| `role` | string | No | organizer, attendee, optional |

**Response 201:**
```json
{
  "data": {
    "id": "participant-uuid-456",
    "type": "participant",
    "attributes": {
      "unique_id": "participant-uuid-456",
      "name": "Bob Wilson",
      "email": "bob@acme.com",
      "role": "attendee",
      "rsvp_status": "pending",
      "created_at": "2026-01-15T10:30:00Z"
    }
  }
}
```

---

### PUT /meetings/:unique_id - Update Meeting

Updates an existing meeting.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/meetings/meeting-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "meeting": {
      "title": "Quarterly Review - Updated",
      "start_time": "2026-03-16T14:00:00Z",
      "end_time": "2026-03-16T15:30:00Z"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "meeting-uuid-123",
    "type": "meeting",
    "attributes": {
      "unique_id": "meeting-uuid-123",
      "title": "Quarterly Review - Updated",
      "start_time": "2026-03-16T14:00:00Z",
      "end_time": "2026-03-16T15:30:00Z",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Meeting not found

---

### DELETE /meetings/:unique_id - Delete Meeting

Deletes a meeting.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/meetings/meeting-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Meeting not found

---

### DELETE /meetings/:unique_id/participants/:participant_unique_id - Remove Participant

Removes a participant from a meeting.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/meetings/meeting-uuid-123/participants/participant-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Meeting
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Meeting title |
| `description` | string | Meeting description |
| `start_time` | timestamp | Start time (ISO 8601) |
| `end_time` | timestamp | End time (ISO 8601) |
| `location` | string | Meeting location |
| `meeting_type` | enum | in_person, virtual, phone |
| `status` | enum | scheduled, in_progress, completed, cancelled |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Participant
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Participant name |
| `email` | string | Participant email |
| `role` | enum | organizer, attendee, optional |
| `rsvp_status` | enum | pending, accepted, declined, tentative |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Meeting Not Found",
    "detail": "The requested meeting could not be found."
  }]
}
```
