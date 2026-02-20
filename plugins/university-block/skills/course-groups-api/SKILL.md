---
name: university-course-groups-api
description: Manage 23blocks University course groups via REST API. Use when creating groups, enrolling students in groups, adding teachers to groups, or retrieving group tests and test responses.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Course Groups API

Complete API reference for 23blocks University course group management with enrollment, teachers, and test tracking.

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
curl -X GET "$BLOCKS_API_URL/course_groups/group-uuid-101/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /course_groups/:unique_id/ - Get Group with Teachers/Students

Retrieves a course group with its assigned teachers and enrolled students.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/course_groups/group-uuid-101/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "group-uuid-101",
    "type": "course_group",
    "attributes": {
      "unique_id": "group-uuid-101",
      "name": "CS101 - Morning Section",
      "course_id": "course-uuid-456",
      "schedule": "MWF 9:00-10:00",
      "max_students": 30,
      "start_date": "2025-09-01",
      "end_date": "2025-12-15",
      "status": "active",
      "created_at": "2025-06-01T10:00:00Z"
    },
    "relationships": {
      "teachers": {
        "data": [
          { "id": "teacher-uuid-789", "type": "teacher" }
        ]
      },
      "students": {
        "data": [
          { "id": "student-uuid-123", "type": "user" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Course group not found

---

### POST /course_groups/ - Create Group

Creates a new course group.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/course_groups/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "course_group": {
      "name": "CS101 - Morning Section",
      "course_id": "course-uuid-456",
      "schedule": "MWF 9:00-10:00",
      "max_students": 30,
      "start_date": "2025-09-01",
      "end_date": "2025-12-15"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Group name |
| `course_id` | uuid | Yes | Parent course ID |
| `schedule` | string | No | Schedule description |
| `max_students` | integer | No | Maximum students in group |
| `start_date` | date | No | Group start date |
| `end_date` | date | No | Group end date |

**Response 201:**
```json
{
  "data": {
    "id": "group-uuid-101",
    "type": "course_group",
    "attributes": {
      "unique_id": "group-uuid-101",
      "name": "CS101 - Morning Section",
      "course_id": "course-uuid-456",
      "schedule": "MWF 9:00-10:00",
      "max_students": 30,
      "start_date": "2025-09-01",
      "end_date": "2025-12-15",
      "status": "active",
      "created_at": "2025-06-01T10:00:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### POST /course_groups/:unique_id/enrollment - Enroll Student

Enrolls a student in a course group.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/course_groups/group-uuid-101/enrollment" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "enrollment": {
      "user_unique_id": "student-uuid-123"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | Student unique ID |

**Response 201:**
```json
{
  "message": "Student enrolled in group successfully"
}
```

**Errors:**
- `422 Unprocessable Entity` - Group is full or student already enrolled

---

### POST /course_groups/:unique_id/teachers - Add Teacher

Assigns a teacher to a course group.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/course_groups/group-uuid-101/teachers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "teacher": {
      "teacher_unique_id": "teacher-uuid-789"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `teacher_unique_id` | uuid | Yes | Teacher unique ID |

**Response 201:**
```json
{
  "message": "Teacher assigned to group successfully"
}
```

---

### GET /course_groups/:unique_id/tests - Get Tests

Lists all tests for a course group.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/course_groups/group-uuid-101/tests" \
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
        "name": "Midterm Exam",
        "test_type": "exam",
        "time_limit": 120,
        "max_score": 100,
        "passing_score": 60,
        "status": "active"
      }
    }
  ]
}
```

---

### GET /course_groups/:unique_id/tests/:test_unique_id/responses - Test Responses

Retrieves all student responses for a specific test in the group.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/course_groups/group-uuid-101/tests/test-uuid-001/responses" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "response-uuid-001",
      "type": "test_response",
      "attributes": {
        "unique_id": "response-uuid-001",
        "user_unique_id": "student-uuid-123",
        "test_unique_id": "test-uuid-001",
        "score": 85,
        "max_score": 100,
        "passed": true,
        "started_at": "2025-10-15T09:00:00Z",
        "finished_at": "2025-10-15T10:30:00Z"
      }
    }
  ],
  "meta": {
    "total_count": 28,
    "average_score": 72.5
  }
}
```

---

## Data Models

### CourseGroup
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Group name |
| `course_id` | uuid | Parent course ID |
| `schedule` | string | Schedule description |
| `max_students` | integer | Maximum students in group |
| `start_date` | date | Group start date |
| `end_date` | date | Group end date |
| `status` | string | Status: `active`, `inactive`, `completed` |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Enrollment Failed",
    "detail": "Course group is at maximum capacity."
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
// CourseGroupsService — client.university.courseGroups
client.university.courseGroups.get(uniqueId: string): Promise<CourseGroup>;
client.university.courseGroups.create(data: CreateCourseGroupRequest): Promise<CourseGroup>;
client.university.courseGroups.addStudent(uniqueId: string, studentUniqueId: string): Promise<CourseGroup>;
client.university.courseGroups.addTeacher(uniqueId: string, teacherUniqueId: string): Promise<CourseGroup>;
client.university.courseGroups.getTests(uniqueId: string): Promise<unknown[]>;
client.university.courseGroups.getTestResponses(uniqueId: string, testUniqueId: string): Promise<unknown[]>;
```

### TypeScript Types

```typescript
import type {
  CourseGroup,
  CreateCourseGroupRequest,
  ListCourseGroupsParams,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.courseGroups.get('group-unique-id');
}
```
