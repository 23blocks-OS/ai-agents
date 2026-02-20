---
name: university-identities-api
description: Manage 23blocks University student identities via REST API. Use when registering students, managing user profiles, handling enrollment, archiving/restoring users, or retrieving enrolled courses, groups, content trees, and notes.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Identities API

Complete API reference for 23blocks University student identity management with enrollment, archiving, and content tree navigation.

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
curl -X GET "$BLOCKS_API_URL/users/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /users/ - List Active Students

Lists all active students with pagination and search.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `search` | string | No | Search by name or email |

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
        "phone": "+1234567890",
        "status": "active",
        "enrollment_date": "2025-09-01T00:00:00Z",
        "created_at": "2025-08-15T10:30:00Z"
      }
    }
  ],
  "meta": {
    "total_count": 150,
    "page": 1,
    "records": 20
  }
}
```

---

### GET /users/status/archive - List Archived Students

Lists all archived students.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/status/archive" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "student-uuid-456",
      "type": "user",
      "attributes": {
        "unique_id": "student-uuid-456",
        "first_name": "John",
        "last_name": "Smith",
        "email": "john.smith@example.com",
        "status": "archived",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /users/:unique_id/ - Get Student Details

Retrieves a single student by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

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
      "email": "jane.doe@example.com",
      "phone": "+1234567890",
      "status": "active",
      "enrollment_date": "2025-09-01T00:00:00Z",
      "created_at": "2025-08-15T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Student not found

---

### POST /users/:unique_id/register/ - Register New User

Registers a new user/student in the university system.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/student-uuid-123/register/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newstudent@example.com",
      "first_name": "Jane",
      "last_name": "Doe",
      "phone": "+1234567890"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Student email |
| `first_name` | string | Yes | First name |
| `last_name` | string | Yes | Last name |
| `phone` | string | No | Phone number |

**Response 201:**
```json
{
  "data": {
    "id": "student-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "student-uuid-123",
      "first_name": "Jane",
      "last_name": "Doe",
      "email": "newstudent@example.com",
      "phone": "+1234567890",
      "status": "active",
      "created_at": "2025-09-01T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (missing email, duplicate user)

---

### PUT /users/:unique_id/ - Update User Profile

Updates an existing student profile.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/student-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "first_name": "Janet",
      "phone": "+0987654321"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |
| `email` | string | No | Email address |
| `phone` | string | No | Phone number |

**Response 200:**
```json
{
  "data": {
    "id": "student-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "student-uuid-123",
      "first_name": "Janet",
      "last_name": "Doe",
      "email": "jane.doe@example.com",
      "phone": "+0987654321",
      "status": "active",
      "created_at": "2025-08-15T10:30:00Z"
    }
  }
}
```

---

### PUT /users/:unique_id/courses - Update User Course Enrollment

Updates the courses a student is enrolled in.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/student-uuid-123/courses" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "courses": ["course-uuid-456", "course-uuid-789"]
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `courses` | array | Yes | Array of course unique IDs |

**Response 200:**
```json
{
  "data": {
    "id": "student-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "student-uuid-123",
      "first_name": "Jane",
      "last_name": "Doe"
    },
    "relationships": {
      "courses": {
        "data": [
          { "id": "course-uuid-456", "type": "course" },
          { "id": "course-uuid-789", "type": "course" }
        ]
      }
    }
  }
}
```

---

### PUT /users/:unique_id/restore - Restore Archived User

Restores a previously archived student.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/student-uuid-456/restore" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "student-uuid-456",
    "type": "user",
    "attributes": {
      "unique_id": "student-uuid-456",
      "status": "active"
    }
  }
}
```

---

### DELETE /users/:unique_id/archive - Archive User

Archives a student (soft delete).

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/student-uuid-123/archive" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "student-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "student-uuid-123",
      "status": "archived"
    }
  }
}
```

---

### GET /users/:unique_id/courses - Get Enrolled Courses

Retrieves all courses a student is enrolled in.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/courses" \
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

### GET /users/:unique_id/availables - Get Available Courses

Retrieves courses available for enrollment (not yet enrolled).

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/availables" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "course-uuid-789",
      "type": "course",
      "attributes": {
        "unique_id": "course-uuid-789",
        "name": "Advanced Algorithms",
        "code": "CS301",
        "level": "advanced",
        "max_students": 25,
        "status": "active"
      }
    }
  ]
}
```

---

### GET /users/:unique_id/groups - Get User's Course Groups

Retrieves all course groups a student belongs to.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/groups" \
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

### GET /users/:unique_id/content_tree/:course_group_unique_id - Content Tree

Retrieves the full content tree (courses, subjects, lessons) for a student in a course group.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/content_tree/group-uuid-101" \
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
              "order": 1,
              "completed": true
            },
            {
              "unique_id": "lesson-uuid-002",
              "name": "Linked Lists",
              "order": 2,
              "completed": false
            }
          ]
        }
      ]
    }
  }
}
```

---

### GET /users/:unique_id/subjects/:subject_unique_id/notes - Student Notes for Subject

Retrieves a student's notes for a specific subject.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/subjects/subject-uuid-001/notes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "note-uuid-001",
      "type": "note",
      "attributes": {
        "unique_id": "note-uuid-001",
        "content": "Arrays store elements in contiguous memory locations",
        "subject_unique_id": "subject-uuid-001",
        "user_unique_id": "student-uuid-123",
        "created_at": "2025-09-15T14:30:00Z"
      }
    }
  ]
}
```

---

## Data Models

### User (Student)
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `email` | string | Email address |
| `phone` | string | Phone number |
| `status` | string | Status: `active`, `archived` |
| `enrollment_date` | timestamp | Date of enrollment |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "User Not Found",
    "detail": "The requested user could not be found."
  }]
}
```
