---
name: university-calendar-api
description: Manage 23blocks university calendar events via REST API. Use for creating, updating, and listing events including course-specific student events with recurrence support.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Calendar API

Complete API reference for 23blocks university calendar event management with course-specific scheduling.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://university.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | University API base URL | `https://university.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/events" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /events/ - List Events

Lists all calendar events.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/events?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Records per page (default: 20) |
| `start_date` | date | No | Filter events starting from (YYYY-MM-DD) |
| `end_date` | date | No | Filter events ending before (YYYY-MM-DD) |
| `event_type` | string | No | Filter by event type |

**Response 200:**
```json
{
  "data": [
    {
      "id": "event-uuid-123",
      "type": "event",
      "attributes": {
        "unique_id": "event-uuid-123",
        "title": "Midterm Exam - Calculus II",
        "description": "Covers chapters 5-8",
        "event_type": "exam",
        "start_time": "2025-03-15T09:00:00Z",
        "end_time": "2025-03-15T11:00:00Z",
        "location": "Hall A, Room 101",
        "recurrence": null,
        "attendees": ["student-uuid-1", "student-uuid-2"],
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 92
  }
}
```

---

### GET /events/:unique_id - Get Event

Retrieves a specific event.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/events/event-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "event-uuid-123",
    "type": "event",
    "attributes": {
      "unique_id": "event-uuid-123",
      "title": "Midterm Exam - Calculus II",
      "description": "Covers chapters 5-8. Bring calculator and ID.",
      "event_type": "exam",
      "start_time": "2025-03-15T09:00:00Z",
      "end_time": "2025-03-15T11:00:00Z",
      "location": "Hall A, Room 101",
      "recurrence": null,
      "attendees": ["student-uuid-1", "student-uuid-2"],
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Event not found

---

### POST /events/ - Create Event

Creates a new calendar event.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/events" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "event": {
      "title": "Weekly Study Group - Physics",
      "description": "Open study group for Physics 101 students",
      "event_type": "study_group",
      "start_time": "2025-02-10T16:00:00Z",
      "end_time": "2025-02-10T18:00:00Z",
      "location": "Library, Study Room 3",
      "recurrence": "weekly",
      "attendees": ["student-uuid-1", "student-uuid-2", "student-uuid-3"],
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | Event title |
| `description` | string | No | Event description |
| `event_type` | string | No | Event type (exam, lecture, study_group, office_hours, deadline, other) |
| `start_time` | datetime | Yes | Start time (ISO 8601) |
| `end_time` | datetime | Yes | End time (ISO 8601) |
| `location` | string | No | Event location |
| `recurrence` | string | No | Recurrence rule (daily, weekly, biweekly, monthly, null) |
| `attendees` | array | No | Array of user unique IDs |
| `status` | enum | No | active, cancelled (default: active) |

**Response 201:**
```json
{
  "data": {
    "id": "event-uuid-new",
    "type": "event",
    "attributes": {
      "unique_id": "event-uuid-new",
      "title": "Weekly Study Group - Physics",
      "description": "Open study group for Physics 101 students",
      "event_type": "study_group",
      "start_time": "2025-02-10T16:00:00Z",
      "end_time": "2025-02-10T18:00:00Z",
      "location": "Library, Study Room 3",
      "recurrence": "weekly",
      "attendees": ["student-uuid-1", "student-uuid-2", "student-uuid-3"],
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /events/:unique_id - Update Event

Updates an existing event.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/events/event-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "event": {
      "title": "Midterm Exam - Calculus II (Rescheduled)",
      "start_time": "2025-03-17T09:00:00Z",
      "end_time": "2025-03-17T11:00:00Z",
      "location": "Hall B, Room 205"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "event-uuid-123",
    "type": "event",
    "attributes": {
      "unique_id": "event-uuid-123",
      "title": "Midterm Exam - Calculus II (Rescheduled)",
      "start_time": "2025-03-17T09:00:00Z",
      "end_time": "2025-03-17T11:00:00Z",
      "location": "Hall B, Room 205",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Event not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /events/:unique_id - Delete Event

Deletes a calendar event.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/events/event-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Course-Specific Student Events

### GET /courses/:unique_id/students/:user_unique_id/events - Student Course Events

Lists all events for a student within a specific course.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/courses/course-uuid-789/students/student-uuid-123/events?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "event-uuid-456",
      "type": "event",
      "attributes": {
        "unique_id": "event-uuid-456",
        "title": "Assignment Due - Problem Set 3",
        "description": "Submit via course portal",
        "event_type": "deadline",
        "start_time": "2025-02-20T23:59:00Z",
        "end_time": "2025-02-20T23:59:00Z",
        "location": null,
        "recurrence": null,
        "status": "active",
        "created_at": "2025-01-15T08:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 18
  }
}
```

---

### GET /courses/:unique_id/students/:user_unique_id/events/:event_unique_id - Specific Student Course Event

Retrieves a specific event for a student within a course.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/courses/course-uuid-789/students/student-uuid-123/events/event-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "event-uuid-456",
    "type": "event",
    "attributes": {
      "unique_id": "event-uuid-456",
      "title": "Assignment Due - Problem Set 3",
      "description": "Submit via course portal",
      "event_type": "deadline",
      "start_time": "2025-02-20T23:59:00Z",
      "end_time": "2025-02-20T23:59:00Z",
      "location": null,
      "recurrence": null,
      "status": "active",
      "created_at": "2025-01-15T08:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Event, course, or student not found

---

### POST /courses/:unique_id/students/:user_unique_id/events - Create Student Course Event

Creates a new event for a student within a specific course.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/courses/course-uuid-789/students/student-uuid-123/events" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "event": {
      "title": "Office Hours - Prof. Johnson",
      "description": "Discuss midterm preparation",
      "event_type": "office_hours",
      "start_time": "2025-02-18T14:00:00Z",
      "end_time": "2025-02-18T15:00:00Z",
      "location": "Faculty Building, Room 302"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "event-uuid-new",
    "type": "event",
    "attributes": {
      "unique_id": "event-uuid-new",
      "title": "Office Hours - Prof. Johnson",
      "description": "Discuss midterm preparation",
      "event_type": "office_hours",
      "start_time": "2025-02-18T14:00:00Z",
      "end_time": "2025-02-18T15:00:00Z",
      "location": "Faculty Building, Room 302",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Course or student not found
- `422 Unprocessable Entity` - Validation errors

---

## Data Models

### Event
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Event title |
| `description` | string | Event description |
| `event_type` | string | exam, lecture, study_group, office_hours, deadline, other |
| `start_time` | datetime | Start time (ISO 8601) |
| `end_time` | datetime | End time (ISO 8601) |
| `location` | string | Event location |
| `recurrence` | string | daily, weekly, biweekly, monthly, null |
| `attendees` | array | Array of user unique IDs |
| `status` | enum | active, cancelled |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Start time must be before end time."
  }]
}
```
