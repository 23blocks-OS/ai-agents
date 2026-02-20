---
name: university-teachers-api
description: Manage 23blocks University teachers via REST API. Use when listing teachers, managing coaching matches and sessions, setting availability, tracking attendance, handling tests, or promoting students.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Teachers API

Complete API reference for 23blocks University teacher management with coaching, availability, attendance, tests, and student promotion.

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
curl -X GET "$BLOCKS_API_URL/teachers/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /teachers/ - List Active Teachers

Lists all active teachers.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/?page=1&records=20" \
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
      "id": "teacher-uuid-789",
      "type": "teacher",
      "attributes": {
        "unique_id": "teacher-uuid-789",
        "first_name": "Dr. Alan",
        "last_name": "Turing",
        "email": "alan.turing@example.com",
        "specialization": "Computer Science",
        "status": "active",
        "created_at": "2025-01-15T10:00:00Z"
      }
    }
  ],
  "meta": {
    "total_count": 12,
    "page": 1,
    "records": 20
  }
}
```

---

### GET /teachers/status/archive - Archived Teachers

Lists all archived teachers.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/status/archive" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "teacher-uuid-archived",
      "type": "teacher",
      "attributes": {
        "unique_id": "teacher-uuid-archived",
        "first_name": "Dr. Ada",
        "last_name": "Lovelace",
        "email": "ada.lovelace@example.com",
        "status": "archived",
        "created_at": "2024-06-01T10:00:00Z"
      }
    }
  ]
}
```

---

### GET /teachers/:unique_id - Get Teacher

Retrieves a single teacher by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "teacher-uuid-789",
    "type": "teacher",
    "attributes": {
      "unique_id": "teacher-uuid-789",
      "first_name": "Dr. Alan",
      "last_name": "Turing",
      "email": "alan.turing@example.com",
      "specialization": "Computer Science",
      "status": "active",
      "created_at": "2025-01-15T10:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Teacher not found

---

### GET /teachers/:unique_id/courses - Teacher Courses

Lists all courses assigned to a teacher.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789/courses" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "course-uuid-456",
      "type": "course",
      "attributes": {
        "unique_id": "course-uuid-456",
        "name": "Introduction to Computer Science",
        "code": "CS101",
        "level": "beginner",
        "status": "active"
      }
    }
  ]
}
```

---

### GET /teachers/:unique_id/groups - Teacher Groups

Lists all course groups assigned to a teacher.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789/groups" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "group-uuid-101",
      "type": "course_group",
      "attributes": {
        "unique_id": "group-uuid-101",
        "name": "CS101 - Morning Section",
        "schedule": "MWF 9:00-10:00",
        "start_date": "2025-09-01",
        "end_date": "2025-12-15",
        "status": "active"
      }
    }
  ]
}
```

---

### GET /teachers/:unique_id/content_tree/:course_group_unique_id - Content Tree

Retrieves the full content tree for a teacher's course group.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789/content_tree/group-uuid-101" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "group-uuid-101",
    "type": "content_tree",
    "attributes": {
      "course_group_unique_id": "group-uuid-101",
      "course": {
        "unique_id": "course-uuid-456",
        "name": "Introduction to Computer Science"
      },
      "subjects": [
        {
          "unique_id": "subject-uuid-001",
          "name": "Data Structures",
          "order": 1,
          "lessons": [
            {
              "unique_id": "lesson-uuid-001",
              "name": "Introduction to Arrays",
              "order": 1
            }
          ]
        }
      ]
    }
  }
}
```

---

### GET /teachers/:unique_id/users/:user_unique_id/content_tree/:course_group_unique_id - Student Tree

Retrieves the content tree for a specific student as viewed by a teacher, including completion status.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789/users/student-uuid-123/content_tree/group-uuid-101" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "group-uuid-101",
    "type": "student_content_tree",
    "attributes": {
      "user_unique_id": "student-uuid-123",
      "course_group_unique_id": "group-uuid-101",
      "course": {
        "unique_id": "course-uuid-456",
        "name": "Introduction to Computer Science"
      },
      "subjects": [
        {
          "unique_id": "subject-uuid-001",
          "name": "Data Structures",
          "lessons": [
            {
              "unique_id": "lesson-uuid-001",
              "name": "Introduction to Arrays",
              "completed": true,
              "completed_at": "2025-09-10T14:30:00Z"
            },
            {
              "unique_id": "lesson-uuid-002",
              "name": "Linked Lists",
              "completed": false
            }
          ]
        }
      ],
      "progress_percentage": 50
    }
  }
}
```

---

## Coaching

### GET /teachers/:unique_id/coaching/active - Active Coaching

Lists all active coaching matches for a teacher.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789/coaching/active" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "match-uuid-001",
      "type": "coaching_match",
      "attributes": {
        "unique_id": "match-uuid-001",
        "teacher_unique_id": "teacher-uuid-789",
        "student_unique_id": "student-uuid-123",
        "status": "active",
        "matched_at": "2025-09-15T10:00:00Z"
      }
    }
  ]
}
```

---

### GET /teachers/:unique_id/coaching/matches - All Matches

Lists all coaching matches for a teacher (active, completed, cancelled).

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789/coaching/matches" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "match-uuid-001",
      "type": "coaching_match",
      "attributes": {
        "unique_id": "match-uuid-001",
        "teacher_unique_id": "teacher-uuid-789",
        "student_unique_id": "student-uuid-123",
        "status": "active",
        "matched_at": "2025-09-15T10:00:00Z"
      }
    },
    {
      "id": "match-uuid-002",
      "type": "coaching_match",
      "attributes": {
        "unique_id": "match-uuid-002",
        "teacher_unique_id": "teacher-uuid-789",
        "student_unique_id": "student-uuid-456",
        "status": "completed",
        "matched_at": "2025-06-01T10:00:00Z"
      }
    }
  ]
}
```

---

### GET /teachers/:unique_id/coaching/available - Available Students

Lists students available for coaching by this teacher.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789/coaching/available" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "student-uuid-789",
      "type": "user",
      "attributes": {
        "unique_id": "student-uuid-789",
        "first_name": "Bob",
        "last_name": "Wilson",
        "email": "bob.wilson@example.com",
        "status": "active"
      }
    }
  ]
}
```

---

### POST /teachers/:unique_id/coachees/find - Find Coachees

Searches for potential coachees matching criteria.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/teachers/teacher-uuid-789/coachees/find" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "filters": {
      "course_id": "course-uuid-456",
      "level": "beginner"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `course_id` | uuid | No | Filter by course |
| `level` | string | No | Filter by student level |

**Response 200:**
```json
{
  "data": [
    {
      "id": "student-uuid-789",
      "type": "user",
      "attributes": {
        "unique_id": "student-uuid-789",
        "first_name": "Bob",
        "last_name": "Wilson",
        "course": "Introduction to Computer Science",
        "level": "beginner"
      }
    }
  ]
}
```

---

## Availability

### GET /teachers/:unique_id/availability - Get Availability

Gets the teacher's availability schedule.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789/availability" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "avail-uuid-001",
      "type": "availability",
      "attributes": {
        "unique_id": "avail-uuid-001",
        "day_of_week": "monday",
        "start_time": "09:00",
        "end_time": "12:00",
        "teacher_unique_id": "teacher-uuid-789"
      }
    },
    {
      "id": "avail-uuid-002",
      "type": "availability",
      "attributes": {
        "unique_id": "avail-uuid-002",
        "day_of_week": "wednesday",
        "start_time": "14:00",
        "end_time": "17:00",
        "teacher_unique_id": "teacher-uuid-789"
      }
    }
  ]
}
```

---

### POST /teachers/:unique_id/availability - Add Availability

Adds a new availability slot for a teacher.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/teachers/teacher-uuid-789/availability" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "availability": {
      "day_of_week": "monday",
      "start_time": "09:00",
      "end_time": "12:00"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `day_of_week` | string | Yes | Day: `monday`, `tuesday`, `wednesday`, `thursday`, `friday`, `saturday`, `sunday` |
| `start_time` | string | Yes | Start time (HH:MM format) |
| `end_time` | string | Yes | End time (HH:MM format) |

**Response 201:**
```json
{
  "data": {
    "id": "avail-uuid-001",
    "type": "availability",
    "attributes": {
      "unique_id": "avail-uuid-001",
      "day_of_week": "monday",
      "start_time": "09:00",
      "end_time": "12:00",
      "created_at": "2025-08-01T10:00:00Z"
    }
  }
}
```

---

### PUT /teachers/:unique_id/availability/:availability_unique_id - Update Availability

Updates an existing availability slot.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/teachers/teacher-uuid-789/availability/avail-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "availability": {
      "start_time": "10:00",
      "end_time": "13:00"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "avail-uuid-001",
    "type": "availability",
    "attributes": {
      "unique_id": "avail-uuid-001",
      "day_of_week": "monday",
      "start_time": "10:00",
      "end_time": "13:00"
    }
  }
}
```

---

### DELETE /teachers/:unique_id/availability/:availability_unique_id - Delete Availability

Deletes a specific availability slot.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/teachers/teacher-uuid-789/availability/avail-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Availability deleted successfully"
}
```

---

### DELETE /teachers/:unique_id/availability - Delete All Availability

Deletes all availability slots for a teacher.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/teachers/teacher-uuid-789/availability" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "All availability slots deleted successfully"
}
```

---

## Coaching Sessions

### GET /teachers/:unique_id/coaching_sessions - Sessions

Lists all coaching sessions for a teacher.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789/coaching_sessions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "session-uuid-001",
      "type": "coaching_session",
      "attributes": {
        "unique_id": "session-uuid-001",
        "teacher_unique_id": "teacher-uuid-789",
        "student_unique_id": "student-uuid-123",
        "scheduled_at": "2025-09-20T10:00:00Z",
        "duration_minutes": 60,
        "status": "scheduled",
        "notes": "Review data structures progress"
      }
    }
  ]
}
```

---

## Teacher Tests

### GET /teachers/:unique_id/tests - Tests

Lists all tests available to a teacher.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789/tests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "test-uuid-001",
      "type": "test",
      "attributes": {
        "unique_id": "test-uuid-001",
        "name": "Teacher Certification: CS Fundamentals",
        "test_type": "certification",
        "time_limit": 90,
        "max_score": 100,
        "status": "active"
      }
    }
  ]
}
```

---

### POST /teachers/:unique_id/test/:test_unique_id - Start Test

Starts a test for a teacher.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/teachers/teacher-uuid-789/test/test-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 201:**
```json
{
  "data": {
    "id": "instance-uuid-301",
    "type": "test_instance",
    "attributes": {
      "unique_id": "instance-uuid-301",
      "test_unique_id": "test-uuid-001",
      "user_unique_id": "teacher-uuid-789",
      "status": "in_progress",
      "started_at": "2025-09-20T10:00:00Z"
    }
  }
}
```

---

### PUT /teachers/:unique_id/test/:test_instance_unique_id - Submit Response

Submits a response to a test question as a teacher.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/teachers/teacher-uuid-789/test/instance-uuid-301" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "response": {
      "question_unique_id": "question-uuid-001",
      "option_unique_id": "option-uuid-001"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `question_unique_id` | uuid | Yes | Question being answered |
| `option_unique_id` | uuid | Yes | Selected option |

**Response 200:**
```json
{
  "data": {
    "id": "instance-uuid-301",
    "type": "test_instance",
    "attributes": {
      "unique_id": "instance-uuid-301",
      "status": "in_progress",
      "responses_count": 5
    }
  }
}
```

---

### PUT /teachers/:unique_id/test/:test_instance_unique_id/finish - Finish Test

Finishes a teacher's test.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/teachers/teacher-uuid-789/test/instance-uuid-301/finish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "instance-uuid-301",
    "type": "test_instance",
    "attributes": {
      "unique_id": "instance-uuid-301",
      "status": "completed",
      "score": 95,
      "max_score": 100,
      "passed": true,
      "finished_at": "2025-09-20T11:15:00Z"
    }
  }
}
```

---

## Attendance

### GET /teachers/:unique_id/attendance - Attendance

Retrieves attendance records for a teacher's sessions.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-789/attendance" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "attendance-uuid-001",
      "type": "attendance",
      "attributes": {
        "unique_id": "attendance-uuid-001",
        "teacher_unique_id": "teacher-uuid-789",
        "user_unique_id": "student-uuid-123",
        "course_group_unique_id": "group-uuid-101",
        "date": "2025-09-15",
        "status": "present",
        "created_at": "2025-09-15T09:05:00Z"
      }
    }
  ]
}
```

---

### POST /teachers/:unique_id/attendance - Register Attendance

Registers attendance for a student.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/teachers/teacher-uuid-789/attendance" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "attendance": {
      "user_unique_id": "student-uuid-123",
      "course_group_unique_id": "group-uuid-101",
      "date": "2025-09-15",
      "status": "present"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | Student unique ID |
| `course_group_unique_id` | uuid | Yes | Course group ID |
| `date` | date | Yes | Date of attendance |
| `status` | string | Yes | Status: `present`, `absent`, `late`, `excused` |

**Response 201:**
```json
{
  "data": {
    "id": "attendance-uuid-001",
    "type": "attendance",
    "attributes": {
      "unique_id": "attendance-uuid-001",
      "user_unique_id": "student-uuid-123",
      "course_group_unique_id": "group-uuid-101",
      "date": "2025-09-15",
      "status": "present",
      "created_at": "2025-09-15T09:05:00Z"
    }
  }
}
```

---

## Student Promotion

### PUT /teachers/:unique_id/users/:user_unique_id/promote - Promote Student

Promotes a student to the next level or course.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/teachers/teacher-uuid-789/users/student-uuid-123/promote" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "promotion": {
      "target_course_id": "course-uuid-advanced",
      "reason": "Completed all requirements with distinction"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `target_course_id` | uuid | No | Target course for promotion |
| `reason` | string | No | Reason for promotion |

**Response 200:**
```json
{
  "data": {
    "id": "student-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "student-uuid-123",
      "first_name": "Jane",
      "last_name": "Doe",
      "promoted": true,
      "promoted_to": "course-uuid-advanced",
      "promoted_at": "2025-12-15T10:00:00Z"
    }
  }
}
```

---

## Data Models

### Teacher
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `email` | string | Email address |
| `specialization` | string | Teaching specialization |
| `status` | string | Status: `active`, `archived` |
| `created_at` | timestamp | Creation time |

### Availability
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `teacher_unique_id` | uuid | Teacher ID |
| `day_of_week` | string | Day of week |
| `start_time` | string | Start time (HH:MM) |
| `end_time` | string | End time (HH:MM) |

### CoachingMatch
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `teacher_unique_id` | uuid | Teacher/coach ID |
| `student_unique_id` | uuid | Student/coachee ID |
| `status` | string | Status: `active`, `completed`, `cancelled` |
| `matched_at` | timestamp | Match time |

### CoachingSession
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `teacher_unique_id` | uuid | Teacher ID |
| `student_unique_id` | uuid | Student ID |
| `scheduled_at` | timestamp | Scheduled time |
| `duration_minutes` | integer | Session duration |
| `status` | string | Status: `scheduled`, `completed`, `cancelled` |
| `notes` | string | Session notes |

### Attendance
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `teacher_unique_id` | uuid | Recording teacher ID |
| `user_unique_id` | uuid | Student ID |
| `course_group_unique_id` | uuid | Course group ID |
| `date` | date | Attendance date |
| `status` | string | Status: `present`, `absent`, `late`, `excused` |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Teacher Not Found",
    "detail": "The requested teacher could not be found."
  }]
}
```
