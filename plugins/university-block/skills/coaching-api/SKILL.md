---
name: university-coaching-api
description: Manage 23blocks university coaching matches and sessions via REST API. Use for creating student-teacher pairings, scheduling coaching sessions, tracking check-ins/check-outs, and managing session notes.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Coaching API

Complete API reference for 23blocks university coaching match management and session scheduling.

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
curl -X GET "$BLOCKS_API_URL/coaching/sessions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Coaching Matches

### POST /matches/ - Create Match

Creates a coaching match (student-teacher pairing).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/matches" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "match": {
      "student_id": "student-uuid-123",
      "teacher_id": "teacher-uuid-456"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `student_id` | uuid | Yes | Student unique ID |
| `teacher_id` | uuid | Yes | Teacher/coach unique ID |

**Response 201:**
```json
{
  "data": {
    "id": "match-uuid-789",
    "type": "match",
    "attributes": {
      "unique_id": "match-uuid-789",
      "student_id": "student-uuid-123",
      "teacher_id": "teacher-uuid-456",
      "status": "pending",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (e.g., duplicate match)

---

### PUT /matches/:unique_id/activate - Activate Match

Activates a pending coaching match.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/matches/match-uuid-789/activate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "match-uuid-789",
    "type": "match",
    "attributes": {
      "unique_id": "match-uuid-789",
      "student_id": "student-uuid-123",
      "teacher_id": "teacher-uuid-456",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Match not found
- `422 Unprocessable Entity` - Invalid state transition

---

### PUT /matches/:unique_id/deactivate - Deactivate Match

Deactivates an active coaching match.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/matches/match-uuid-789/deactivate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "match-uuid-789",
    "type": "match",
    "attributes": {
      "unique_id": "match-uuid-789",
      "status": "inactive",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Match not found
- `422 Unprocessable Entity` - Invalid state transition

---

### DELETE /matches/:unique_id - Delete Match

Deletes a coaching match.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/matches/match-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### POST /matches/evaluate - Evaluate Potential Matches

Evaluates potential student-teacher matches based on criteria.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/matches/evaluate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": "student-uuid-123",
    "criteria": {
      "subject": "mathematics",
      "availability": "weekdays"
    }
  }'
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "teacher-uuid-456",
      "type": "match_evaluation",
      "attributes": {
        "teacher_id": "teacher-uuid-456",
        "compatibility_score": 0.92,
        "availability_overlap": 12,
        "subjects_matched": ["mathematics", "statistics"]
      }
    }
  ]
}
```

---

### POST /matches/availabilities - Evaluate Availability Compatibility

Evaluates availability compatibility between student and teacher.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/matches/availabilities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": "student-uuid-123",
    "teacher_id": "teacher-uuid-456"
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "availability_evaluation",
    "attributes": {
      "student_id": "student-uuid-123",
      "teacher_id": "teacher-uuid-456",
      "compatible": true,
      "overlapping_slots": [
        {
          "day_of_week": "monday",
          "start_time": "10:00",
          "end_time": "12:00"
        },
        {
          "day_of_week": "wednesday",
          "start_time": "14:00",
          "end_time": "16:00"
        }
      ]
    }
  }
}
```

---

### GET /users/:unique_id/coaching/active - Student Active Match

Gets the student's currently active coaching match.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/coaching/active" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "match-uuid-789",
    "type": "match",
    "attributes": {
      "unique_id": "match-uuid-789",
      "student_id": "student-uuid-123",
      "teacher_id": "teacher-uuid-456",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - No active match found

---

### GET /users/:unique_id/coaching/matches - Student All Matches

Lists all coaching matches for a student.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/coaching/matches" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "match-uuid-789",
      "type": "match",
      "attributes": {
        "unique_id": "match-uuid-789",
        "student_id": "student-uuid-123",
        "teacher_id": "teacher-uuid-456",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "match-uuid-111",
      "type": "match",
      "attributes": {
        "unique_id": "match-uuid-111",
        "student_id": "student-uuid-123",
        "teacher_id": "teacher-uuid-222",
        "status": "inactive",
        "created_at": "2024-09-01T08:00:00Z"
      }
    }
  ]
}
```

---

### GET /users/:unique_id/coaching/available - Available Coaches

Lists available coaches for a student.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/coaching/available" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "teacher-uuid-456",
      "type": "teacher",
      "attributes": {
        "unique_id": "teacher-uuid-456",
        "name": "Dr. Smith",
        "subjects": ["mathematics", "statistics"],
        "availability_slots": 8,
        "active_students": 3
      }
    }
  ]
}
```

---

### POST /users/:unique_id/coaches/find - Find Coaches

Searches for coaches matching specific criteria.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/student-uuid-123/coaches/find" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "criteria": {
      "subject": "mathematics",
      "day_of_week": "monday",
      "time_range": "morning"
    }
  }'
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "teacher-uuid-456",
      "type": "teacher",
      "attributes": {
        "unique_id": "teacher-uuid-456",
        "name": "Dr. Smith",
        "subjects": ["mathematics"],
        "matching_slots": [
          {
            "day_of_week": "monday",
            "start_time": "09:00",
            "end_time": "12:00"
          }
        ]
      }
    }
  ]
}
```

---

## Coaching Sessions

### GET /coaching/sessions/ - List Sessions

Lists coaching sessions with pagination and date filters.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/coaching/sessions?page=1&records=20&start_date=2025-01-01&end_date=2025-01-31" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Records per page (default: 20) |
| `start_date` | date | No | Filter by start date (YYYY-MM-DD) |
| `end_date` | date | No | Filter by end date (YYYY-MM-DD) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "session-uuid-123",
      "type": "coaching_session",
      "attributes": {
        "unique_id": "session-uuid-123",
        "match_id": "match-uuid-789",
        "scheduled_at": "2025-01-15T10:00:00Z",
        "duration": 60,
        "location": "Room 201",
        "student_status": "confirmed",
        "teacher_status": "confirmed",
        "notes": null,
        "status": "scheduled",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 95
  }
}
```

---

### GET /coaching/sessions/:unique_id - Get Session

Retrieves a specific coaching session.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/coaching/sessions/session-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "coaching_session",
    "attributes": {
      "unique_id": "session-uuid-123",
      "match_id": "match-uuid-789",
      "scheduled_at": "2025-01-15T10:00:00Z",
      "duration": 60,
      "location": "Room 201",
      "student_status": "confirmed",
      "teacher_status": "confirmed",
      "notes": "Review calculus chapter 5",
      "status": "scheduled",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Session not found

---

### POST /coaching/sessions - Create Session

Creates a new coaching session.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/coaching/sessions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "coaching_session": {
      "match_id": "match-uuid-789",
      "scheduled_at": "2025-01-20T14:00:00Z",
      "duration": 60,
      "location": "Room 201",
      "notes": "Focus on linear algebra"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `match_id` | uuid | Yes | Coaching match ID |
| `scheduled_at` | datetime | Yes | Session date/time (ISO 8601) |
| `duration` | integer | No | Duration in minutes (default: 60) |
| `location` | string | No | Session location |
| `notes` | string | No | Session notes |

**Response 201:**
```json
{
  "data": {
    "id": "session-uuid-456",
    "type": "coaching_session",
    "attributes": {
      "unique_id": "session-uuid-456",
      "match_id": "match-uuid-789",
      "scheduled_at": "2025-01-20T14:00:00Z",
      "duration": 60,
      "location": "Room 201",
      "student_status": "pending",
      "teacher_status": "pending",
      "notes": "Focus on linear algebra",
      "status": "scheduled",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors
- `404 Not Found` - Match not found

---

### PUT /coaching/sessions/:unique_id - Update Session

Updates a coaching session.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/coaching/sessions/session-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "coaching_session": {
      "scheduled_at": "2025-01-21T14:00:00Z",
      "location": "Room 305"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-456",
    "type": "coaching_session",
    "attributes": {
      "unique_id": "session-uuid-456",
      "scheduled_at": "2025-01-21T14:00:00Z",
      "location": "Room 305",
      "status": "scheduled",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /coaching/sessions/:unique_id - Cancel Session

Cancels a coaching session.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/coaching/sessions/session-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /coaching/sessions/:unique_id/students/confirmation - Student Confirm

Student confirms attendance for a session.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/coaching/sessions/session-uuid-123/students/confirmation" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "coaching_session",
    "attributes": {
      "unique_id": "session-uuid-123",
      "student_status": "confirmed",
      "status": "confirmed"
    }
  }
}
```

---

### PUT /coaching/sessions/:unique_id/students/checking - Student Check-in

Student checks in to a session.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/coaching/sessions/session-uuid-123/students/checking" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "coaching_session",
    "attributes": {
      "unique_id": "session-uuid-123",
      "student_status": "checked_in",
      "status": "in_progress"
    }
  }
}
```

---

### PUT /coaching/sessions/:unique_id/students/checkout - Student Check-out

Student checks out of a session.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/coaching/sessions/session-uuid-123/students/checkout" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "coaching_session",
    "attributes": {
      "unique_id": "session-uuid-123",
      "student_status": "checked_out",
      "status": "completed"
    }
  }
}
```

---

### PUT /coaching/sessions/:unique_id/teachers/confirmation - Teacher Confirm

Teacher confirms a coaching session.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/coaching/sessions/session-uuid-123/teachers/confirmation" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "coaching_session",
    "attributes": {
      "unique_id": "session-uuid-123",
      "teacher_status": "confirmed",
      "status": "confirmed"
    }
  }
}
```

---

### PUT /coaching/sessions/:unique_id/teachers/checking - Teacher Check-in

Teacher checks in to a session.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/coaching/sessions/session-uuid-123/teachers/checking" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "coaching_session",
    "attributes": {
      "unique_id": "session-uuid-123",
      "teacher_status": "checked_in",
      "status": "in_progress"
    }
  }
}
```

---

### PUT /coaching/sessions/:unique_id/teachers/checkout - Teacher Check-out

Teacher checks out of a session.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/coaching/sessions/session-uuid-123/teachers/checkout" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "coaching_session",
    "attributes": {
      "unique_id": "session-uuid-123",
      "teacher_status": "checked_out",
      "status": "completed"
    }
  }
}
```

---

### PUT /coaching/sessions/:unique_id/students/notes - Student Notes

Adds or updates student notes for a session.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/coaching/sessions/session-uuid-123/students/notes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "notes": "Great session. Need to review integration techniques."
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "coaching_session",
    "attributes": {
      "unique_id": "session-uuid-123",
      "notes": "Great session. Need to review integration techniques."
    }
  }
}
```

---

### PUT /coaching/sessions/:unique_id/admin/notes - Admin Notes

Adds or updates admin notes for a session.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/coaching/sessions/session-uuid-123/admin/notes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "notes": "Session rescheduled from original date due to holiday."
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-123",
    "type": "coaching_session",
    "attributes": {
      "unique_id": "session-uuid-123",
      "admin_notes": "Session rescheduled from original date due to holiday."
    }
  }
}
```

---

### GET /users/:unique_id/coaching_sessions - Student Sessions

Lists all coaching sessions for a student.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/coaching_sessions?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "session-uuid-123",
      "type": "coaching_session",
      "attributes": {
        "unique_id": "session-uuid-123",
        "match_id": "match-uuid-789",
        "scheduled_at": "2025-01-15T10:00:00Z",
        "duration": 60,
        "location": "Room 201",
        "student_status": "confirmed",
        "teacher_status": "confirmed",
        "status": "scheduled",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 48
  }
}
```

---

### GET /teachers/:unique_id/coaching_sessions - Teacher Sessions

Lists all coaching sessions for a teacher.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-456/coaching_sessions?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "session-uuid-123",
      "type": "coaching_session",
      "attributes": {
        "unique_id": "session-uuid-123",
        "match_id": "match-uuid-789",
        "scheduled_at": "2025-01-15T10:00:00Z",
        "duration": 60,
        "location": "Room 201",
        "student_status": "confirmed",
        "teacher_status": "confirmed",
        "status": "scheduled",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 35
  }
}
```

---

## Data Models

### Match
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `student_id` | uuid | Student unique ID |
| `teacher_id` | uuid | Teacher/coach unique ID |
| `status` | enum | pending, active, inactive |
| `created_at` | timestamp | Creation time |

### CoachingSession
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `match_id` | uuid | Associated match ID |
| `scheduled_at` | datetime | Scheduled date/time |
| `duration` | integer | Duration in minutes |
| `location` | string | Session location |
| `student_status` | enum | pending, confirmed, checked_in, checked_out |
| `teacher_status` | enum | pending, confirmed, checked_in, checked_out |
| `notes` | text | Session notes |
| `admin_notes` | text | Admin notes |
| `status` | enum | scheduled, confirmed, in_progress, completed, cancelled |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Match already exists for this student-teacher pair."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-university
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
// CoachingSessionsService — client.university.coachingSessions
client.university.coachingSessions.list(params?: ListCoachingSessionsParams): Promise<PageResult<CoachingSession>>;
client.university.coachingSessions.create(data: CreateCoachingSessionRequest): Promise<CoachingSession>;
client.university.coachingSessions.update(uniqueId: string, data: UpdateCoachingSessionRequest): Promise<CoachingSession>;
client.university.coachingSessions.delete(uniqueId: string): Promise<void>;
client.university.coachingSessions.getByStudent(studentUniqueId: string): Promise<CoachingSession[]>;
client.university.coachingSessions.getByTeacher(teacherUniqueId: string): Promise<CoachingSession[]>;
client.university.coachingSessions.studentConfirm(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.studentCheckIn(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.studentCheckOut(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.studentNotes(uniqueId: string, notes: string): Promise<CoachingSession>;
client.university.coachingSessions.teacherConfirm(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.teacherCheckIn(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.teacherCheckOut(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.adminNotes(uniqueId: string, notes: string): Promise<CoachingSession>;
```

### TypeScript Types

```typescript
import type {
  CoachingSession,
  CreateCoachingSessionRequest,
  UpdateCoachingSessionRequest,
  ListCoachingSessionsParams,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.coachingSessions.list();
}
```
