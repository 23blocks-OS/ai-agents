---
name: university-lessons-api
description: Manage 23blocks University lessons via REST API. Use when creating, updating lessons, marking completion, managing resources, or retrieving lesson tests.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Lessons API

Complete API reference for 23blocks University lesson management with resources, completion tracking, and tests.

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
curl -X GET "$BLOCKS_API_URL/lessons/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /lessons/ - List Lessons

Lists all lessons.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/lessons/?page=1&records=20" \
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
      "id": "lesson-uuid-001",
      "type": "lesson",
      "attributes": {
        "unique_id": "lesson-uuid-001",
        "name": "Introduction to Arrays",
        "description": "Understanding array data structures",
        "subject_id": "subject-uuid-001",
        "order": 1,
        "duration_minutes": 45,
        "content_type": "lecture",
        "status": "active",
        "created_at": "2025-06-15T10:00:00Z"
      }
    }
  ],
  "meta": {
    "total_count": 48,
    "page": 1,
    "records": 20
  }
}
```

---

### GET /lessons/:unique_id/ - Get Lesson with Resources

Retrieves a single lesson with its resources.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/lessons/lesson-uuid-001/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "lesson-uuid-001",
    "type": "lesson",
    "attributes": {
      "unique_id": "lesson-uuid-001",
      "name": "Introduction to Arrays",
      "description": "Understanding array data structures",
      "subject_id": "subject-uuid-001",
      "order": 1,
      "duration_minutes": 45,
      "content_type": "lecture",
      "status": "active",
      "created_at": "2025-06-15T10:00:00Z"
    },
    "relationships": {
      "resources": {
        "data": [
          { "id": "resource-uuid-003", "type": "resource" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Lesson not found

---

### POST /lessons/ - Create Lesson

Creates a new lesson.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/lessons/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "lesson": {
      "name": "Introduction to Arrays",
      "description": "Understanding array data structures",
      "subject_id": "subject-uuid-001",
      "order": 1,
      "duration_minutes": 45,
      "content_type": "lecture"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Lesson name |
| `description` | string | No | Lesson description |
| `subject_id` | uuid | Yes | Parent subject ID |
| `order` | integer | No | Display order within subject |
| `duration_minutes` | integer | No | Lesson duration in minutes |
| `content_type` | string | No | Type: `lecture`, `lab`, `workshop`, `quiz` |

**Response 201:**
```json
{
  "data": {
    "id": "lesson-uuid-001",
    "type": "lesson",
    "attributes": {
      "unique_id": "lesson-uuid-001",
      "name": "Introduction to Arrays",
      "description": "Understanding array data structures",
      "subject_id": "subject-uuid-001",
      "order": 1,
      "duration_minutes": 45,
      "content_type": "lecture",
      "status": "active",
      "created_at": "2025-06-15T10:00:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /lessons/:unique_id - Update Lesson

Updates an existing lesson.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/lessons/lesson-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "lesson": {
      "description": "Deep dive into array operations and complexity",
      "duration_minutes": 60
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "lesson-uuid-001",
    "type": "lesson",
    "attributes": {
      "unique_id": "lesson-uuid-001",
      "name": "Introduction to Arrays",
      "description": "Deep dive into array operations and complexity",
      "duration_minutes": 60,
      "status": "active"
    }
  }
}
```

---

### PUT /lessons/:unique_id/completed - Mark Completed

Marks a lesson as completed for the current user.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/lessons/lesson-uuid-001/completed" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "lesson-uuid-001",
    "type": "lesson",
    "attributes": {
      "unique_id": "lesson-uuid-001",
      "name": "Introduction to Arrays",
      "completed": true,
      "completed_at": "2025-09-10T14:30:00Z"
    }
  }
}
```

---

### GET /lessons/:unique_id/resources - Resources

Gets all resources for a lesson.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/lessons/lesson-uuid-001/resources" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "resource-uuid-003",
      "type": "resource",
      "attributes": {
        "unique_id": "resource-uuid-003",
        "name": "Array Operations Slides",
        "file_url": "https://storage.example.com/arrays-slides.pdf",
        "file_type": "application/pdf",
        "created_at": "2025-06-15T10:00:00Z"
      }
    }
  ]
}
```

---

### POST /lessons/:unique_id/resources - Add Resource

Adds a resource to a lesson.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/lessons/lesson-uuid-001/resources" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "resource": {
      "name": "Array Operations Slides",
      "file_url": "https://storage.example.com/arrays-slides.pdf",
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
    "id": "resource-uuid-003",
    "type": "resource",
    "attributes": {
      "unique_id": "resource-uuid-003",
      "name": "Array Operations Slides",
      "file_url": "https://storage.example.com/arrays-slides.pdf",
      "created_at": "2025-06-15T10:00:00Z"
    }
  }
}
```

---

### PUT /lessons/:unique_id/resources/:resource_unique_id - Update Resource

Updates a lesson resource.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/lessons/lesson-uuid-001/resources/resource-uuid-003" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "resource": {
      "name": "Updated Array Slides v2"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "resource-uuid-003",
    "type": "resource",
    "attributes": {
      "unique_id": "resource-uuid-003",
      "name": "Updated Array Slides v2"
    }
  }
}
```

---

### DELETE /lessons/:unique_id/resources/:resource_unique_id - Delete Resource

Deletes a lesson resource.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/lessons/lesson-uuid-001/resources/resource-uuid-003" \
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

### PUT /lessons/:unique_id/presign_upload - Presign Upload

Generates a presigned URL for file upload to a lesson.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/lessons/lesson-uuid-001/presign_upload" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "filename": "exercise-handout.pdf",
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
    "presigned_url": "https://storage.example.com/upload?signature=ghi789",
    "file_url": "https://storage.example.com/lessons/lesson-uuid-001/exercise-handout.pdf",
    "expires_at": "2025-09-01T11:00:00Z"
  }
}
```

---

### GET /lessons/:unique_id/tests - Tests

Gets all content tests for a lesson.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/lessons/lesson-uuid-001/tests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "test-uuid-003",
      "type": "test",
      "attributes": {
        "unique_id": "test-uuid-003",
        "name": "Arrays Practice Quiz",
        "test_type": "quiz",
        "time_limit": 15,
        "max_score": 20,
        "passing_score": 12,
        "status": "active"
      }
    }
  ]
}
```

---

## Data Models

### Lesson
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Lesson name |
| `description` | string | Lesson description |
| `subject_id` | uuid | Parent subject ID |
| `order` | integer | Display order within subject |
| `duration_minutes` | integer | Lesson duration in minutes |
| `content_type` | string | Type: `lecture`, `lab`, `workshop`, `quiz` |
| `status` | string | Status: `active`, `inactive`, `draft` |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Lesson Not Found",
    "detail": "The requested lesson could not be found."
  }]
}
```
