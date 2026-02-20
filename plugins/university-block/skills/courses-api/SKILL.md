---
name: university-courses-api
description: Manage 23blocks University courses via REST API. Use when creating, updating courses, managing enrollment, teachers, groups, resources, assignments, tests, and placement tests.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Courses API

Complete API reference for 23blocks University course management with enrollment, resources, assignments, and assessments.

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
curl -X GET "$BLOCKS_API_URL/courses/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /courses/ - List Courses

Lists all courses.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/courses/?page=1&records=20" \
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
      "id": "course-uuid-456",
      "type": "course",
      "attributes": {
        "unique_id": "course-uuid-456",
        "name": "Introduction to Computer Science",
        "description": "Fundamentals of CS",
        "code": "CS101",
        "duration": "16 weeks",
        "level": "beginner",
        "max_students": 30,
        "status": "active",
        "created_at": "2025-06-01T10:00:00Z"
      }
    }
  ],
  "meta": {
    "total_count": 25,
    "page": 1,
    "records": 20
  }
}
```

---

### GET /courses/:unique_id/ - Get Course with Subjects and Resources

Retrieves a single course with its subjects and resources.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/courses/course-uuid-456/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "course-uuid-456",
    "type": "course",
    "attributes": {
      "unique_id": "course-uuid-456",
      "name": "Introduction to Computer Science",
      "description": "Fundamentals of CS",
      "code": "CS101",
      "duration": "16 weeks",
      "level": "beginner",
      "max_students": 30,
      "status": "active",
      "created_at": "2025-06-01T10:00:00Z"
    },
    "relationships": {
      "subjects": {
        "data": [
          { "id": "subject-uuid-001", "type": "subject" }
        ]
      },
      "resources": {
        "data": [
          { "id": "resource-uuid-001", "type": "resource" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Course not found

---

### POST /courses/ - Create Course

Creates a new course.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/courses/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "course": {
      "name": "Introduction to Computer Science",
      "description": "Fundamentals of CS",
      "code": "CS101",
      "duration": "16 weeks",
      "level": "beginner",
      "max_students": 30
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Course name |
| `description` | string | No | Course description |
| `code` | string | No | Course code (e.g., CS101) |
| `duration` | string | No | Course duration |
| `level` | string | No | Course level (beginner, intermediate, advanced) |
| `max_students` | integer | No | Maximum students allowed |

**Response 201:**
```json
{
  "data": {
    "id": "course-uuid-456",
    "type": "course",
    "attributes": {
      "unique_id": "course-uuid-456",
      "name": "Introduction to Computer Science",
      "description": "Fundamentals of CS",
      "code": "CS101",
      "duration": "16 weeks",
      "level": "beginner",
      "max_students": 30,
      "status": "active",
      "created_at": "2025-06-01T10:00:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /courses/:unique_id - Update Course

Updates an existing course.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/courses/course-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "course": {
      "description": "Updated description for CS fundamentals",
      "max_students": 35
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "course-uuid-456",
    "type": "course",
    "attributes": {
      "unique_id": "course-uuid-456",
      "name": "Introduction to Computer Science",
      "description": "Updated description for CS fundamentals",
      "max_students": 35,
      "status": "active"
    }
  }
}
```

---

### GET /courses/:unique_id/teachers - List Course Teachers

Lists all teachers assigned to a course.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/courses/course-uuid-456/teachers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

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
        "specialization": "Computer Science"
      }
    }
  ]
}
```

---

### GET /courses/:unique_id/students - List Course Students

Lists all students enrolled in a course.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/courses/course-uuid-456/students" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "student-uuid-123",
      "type": "user",
      "attributes": {
        "unique_id": "student-uuid-123",
        "first_name": "Jane",
        "last_name": "Doe",
        "email": "jane.doe@example.com",
        "status": "active"
      }
    }
  ],
  "meta": {
    "total_count": 28
  }
}
```

---

### POST /courses/:unique_id/enrollment - Enroll Student

Enrolls a student in a course.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/courses/course-uuid-456/enrollment" \
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
  "data": {
    "id": "enrollment-uuid-001",
    "type": "enrollment",
    "attributes": {
      "user_unique_id": "student-uuid-123",
      "course_unique_id": "course-uuid-456",
      "enrollment_code": "ENR-2025-001",
      "status": "pending",
      "created_at": "2025-09-01T10:00:00Z"
    }
  }
}
```

---

### POST /courses/:unique_id/teacher - Add Teacher

Assigns a teacher to a course.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/courses/course-uuid-456/teacher" \
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
  "message": "Teacher assigned successfully"
}
```

---

### PUT /courses/:unique_id/enrollment/:enrollment_code - Validate Enrollment

Validates an enrollment code to confirm student enrollment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/courses/course-uuid-456/enrollment/ENR-2025-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "enrollment-uuid-001",
    "type": "enrollment",
    "attributes": {
      "enrollment_code": "ENR-2025-001",
      "status": "validated",
      "validated_at": "2025-09-02T08:00:00Z"
    }
  }
}
```

---

### GET /courses/:unique_id/groups - List Course Groups

Lists all groups for a course.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/courses/course-uuid-456/groups" \
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
        "max_students": 30,
        "start_date": "2025-09-01",
        "end_date": "2025-12-15",
        "status": "active"
      }
    }
  ]
}
```

---

### POST /course/:unique_id/course_groups/enrollment - Register Student to Group

Registers a student into a specific course group.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/course/course-uuid-456/course_groups/enrollment" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "enrollment": {
      "user_unique_id": "student-uuid-123",
      "course_group_unique_id": "group-uuid-101"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | Student unique ID |
| `course_group_unique_id` | uuid | Yes | Course group unique ID |

**Response 201:**
```json
{
  "message": "Student registered to group successfully"
}
```

---

### GET /courses/:unique_id/resources - Get Resources

Gets all resources for a course.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/courses/course-uuid-456/resources" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "resource-uuid-001",
      "type": "resource",
      "attributes": {
        "unique_id": "resource-uuid-001",
        "name": "Course Syllabus",
        "file_url": "https://storage.example.com/syllabus.pdf",
        "file_type": "application/pdf",
        "created_at": "2025-06-15T10:00:00Z"
      }
    }
  ]
}
```

---

### POST /courses/:unique_id/resources - Add Resource

Adds a resource to a course.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/courses/course-uuid-456/resources" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "resource": {
      "name": "Course Syllabus",
      "file_url": "https://storage.example.com/syllabus.pdf",
      "file_type": "application/pdf"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Resource name |
| `file_url` | string | Yes | URL to the file |
| `file_type` | string | No | MIME type of file |

**Response 201:**
```json
{
  "data": {
    "id": "resource-uuid-001",
    "type": "resource",
    "attributes": {
      "unique_id": "resource-uuid-001",
      "name": "Course Syllabus",
      "file_url": "https://storage.example.com/syllabus.pdf",
      "created_at": "2025-06-15T10:00:00Z"
    }
  }
}
```

---

### PUT /courses/:unique_id/resources/:resource_unique_id - Update Resource

Updates a course resource.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/courses/course-uuid-456/resources/resource-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "resource": {
      "name": "Updated Course Syllabus"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "resource-uuid-001",
    "type": "resource",
    "attributes": {
      "unique_id": "resource-uuid-001",
      "name": "Updated Course Syllabus"
    }
  }
}
```

---

### DELETE /courses/:unique_id/resources/:resource_unique_id - Delete Resource

Deletes a course resource.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/courses/course-uuid-456/resources/resource-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Resource deleted successfully"
}
```

---

### PUT /courses/:unique_id/presign_upload - Presign Upload URL

Generates a presigned URL for file upload to a course.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/courses/course-uuid-456/presign_upload" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "filename": "lecture-notes.pdf",
      "content_type": "application/pdf"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filename` | string | Yes | Name of the file |
| `content_type` | string | Yes | MIME type of the file |

**Response 200:**
```json
{
  "data": {
    "presigned_url": "https://storage.example.com/upload?signature=abc123",
    "file_url": "https://storage.example.com/courses/course-uuid-456/lecture-notes.pdf",
    "expires_at": "2025-09-01T11:00:00Z"
  }
}
```

---

### GET /courses/:unique_id/assignments/ - Course Assignments

Lists all assignments for a course.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/courses/course-uuid-456/assignments/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "assignment-uuid-001",
      "type": "assignment",
      "attributes": {
        "unique_id": "assignment-uuid-001",
        "title": "Homework 1: Arrays",
        "description": "Complete exercises on array operations",
        "due_date": "2025-09-15T23:59:00Z",
        "max_score": 100,
        "assignment_type": "homework",
        "status": "active"
      }
    }
  ]
}
```

---

### POST /courses/:unique_id/assignments - Assign to Course

Creates an assignment for a course.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/courses/course-uuid-456/assignments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "assignment": {
      "title": "Homework 1: Arrays",
      "description": "Complete exercises on array operations",
      "due_date": "2025-09-15T23:59:00Z",
      "max_score": 100,
      "assignment_type": "homework"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "assignment-uuid-001",
    "type": "assignment",
    "attributes": {
      "unique_id": "assignment-uuid-001",
      "title": "Homework 1: Arrays",
      "course_unique_id": "course-uuid-456",
      "status": "active",
      "created_at": "2025-09-01T10:00:00Z"
    }
  }
}
```

---

### GET /courses/:unique_id/tests - Course Tests

Lists all content tests for a course.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/courses/course-uuid-456/tests" \
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

### GET /courses/:unique_id/placement - Course Placement Tests

Lists placement tests for a course.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/courses/course-uuid-456/placement" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "placement-uuid-001",
      "type": "placement",
      "attributes": {
        "unique_id": "placement-uuid-001",
        "name": "CS Placement Assessment",
        "description": "Determines student starting level",
        "time_limit": 60,
        "status": "active"
      }
    }
  ]
}
```

---

### POST /courses/:unique_id/placement/ - Create Placement Test for Course

Creates a placement test associated with a course.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/courses/course-uuid-456/placement/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "placement": {
      "name": "CS Placement Assessment",
      "description": "Determines student starting level",
      "time_limit": 60
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Placement test name |
| `description` | string | No | Test description |
| `time_limit` | integer | No | Time limit in minutes |

**Response 201:**
```json
{
  "data": {
    "id": "placement-uuid-001",
    "type": "placement",
    "attributes": {
      "unique_id": "placement-uuid-001",
      "name": "CS Placement Assessment",
      "course_unique_id": "course-uuid-456",
      "status": "active",
      "created_at": "2025-06-15T10:00:00Z"
    }
  }
}
```

---

## Data Models

### Course
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Course name |
| `description` | string | Course description |
| `code` | string | Course code (e.g., CS101) |
| `duration` | string | Course duration |
| `level` | string | Level: `beginner`, `intermediate`, `advanced` |
| `max_students` | integer | Maximum enrollment |
| `status` | string | Status: `active`, `inactive`, `draft` |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Failed",
    "detail": "Name can't be blank."
  }]
}
```
