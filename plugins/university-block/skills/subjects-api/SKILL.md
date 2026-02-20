---
name: university-subjects-api
description: Manage 23blocks University subjects via REST API. Use when creating, updating subjects, adding lessons, managing resources, or retrieving subject tests.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Subjects API

Complete API reference for 23blocks University subject management with lessons, resources, and tests.

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
curl -X GET "$BLOCKS_API_URL/subjects/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /subjects/ - List Subjects

Lists all subjects.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subjects/?page=1&records=20" \
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
      "id": "subject-uuid-001",
      "type": "subject",
      "attributes": {
        "unique_id": "subject-uuid-001",
        "name": "Data Structures",
        "description": "Arrays, lists, trees, and graphs",
        "course_id": "course-uuid-456",
        "order": 1,
        "status": "active",
        "created_at": "2025-06-10T10:00:00Z"
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

### GET /subjects/:unique_id - Get Subject with Lessons

Retrieves a single subject with its lessons.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subjects/subject-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "subject-uuid-001",
    "type": "subject",
    "attributes": {
      "unique_id": "subject-uuid-001",
      "name": "Data Structures",
      "description": "Arrays, lists, trees, and graphs",
      "course_id": "course-uuid-456",
      "order": 1,
      "status": "active",
      "created_at": "2025-06-10T10:00:00Z"
    },
    "relationships": {
      "lessons": {
        "data": [
          { "id": "lesson-uuid-001", "type": "lesson" },
          { "id": "lesson-uuid-002", "type": "lesson" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Subject not found

---

### POST /subjects/ - Create Subject

Creates a new subject.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/subjects/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": {
      "name": "Data Structures",
      "description": "Arrays, lists, trees, and graphs",
      "course_id": "course-uuid-456",
      "order": 1
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Subject name |
| `description` | string | No | Subject description |
| `course_id` | uuid | Yes | Parent course ID |
| `order` | integer | No | Display order within course |

**Response 201:**
```json
{
  "data": {
    "id": "subject-uuid-001",
    "type": "subject",
    "attributes": {
      "unique_id": "subject-uuid-001",
      "name": "Data Structures",
      "description": "Arrays, lists, trees, and graphs",
      "course_id": "course-uuid-456",
      "order": 1,
      "status": "active",
      "created_at": "2025-06-10T10:00:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /subjects/:unique_id - Update Subject

Updates an existing subject.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/subjects/subject-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": {
      "description": "Updated description covering data structures",
      "order": 2
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "subject-uuid-001",
    "type": "subject",
    "attributes": {
      "unique_id": "subject-uuid-001",
      "name": "Data Structures",
      "description": "Updated description covering data structures",
      "order": 2,
      "status": "active"
    }
  }
}
```

---

### POST /subjects/:unique_id/lessons - Add Lesson

Adds a lesson to a subject.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/subjects/subject-uuid-001/lessons" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "lesson": {
      "name": "Introduction to Arrays",
      "description": "Understanding array data structures",
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

---

### GET /subjects/:unique_id/resources - Get Resources

Gets all resources for a subject.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subjects/subject-uuid-001/resources" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "resource-uuid-002",
      "type": "resource",
      "attributes": {
        "unique_id": "resource-uuid-002",
        "name": "Data Structures Textbook Chapter",
        "file_url": "https://storage.example.com/ds-chapter.pdf",
        "file_type": "application/pdf",
        "created_at": "2025-06-15T10:00:00Z"
      }
    }
  ]
}
```

---

### POST /subjects/:unique_id/resources - Add Resource

Adds a resource to a subject.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/subjects/subject-uuid-001/resources" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "resource": {
      "name": "Data Structures Textbook Chapter",
      "file_url": "https://storage.example.com/ds-chapter.pdf",
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
    "id": "resource-uuid-002",
    "type": "resource",
    "attributes": {
      "unique_id": "resource-uuid-002",
      "name": "Data Structures Textbook Chapter",
      "file_url": "https://storage.example.com/ds-chapter.pdf",
      "created_at": "2025-06-15T10:00:00Z"
    }
  }
}
```

---

### PUT /subjects/:unique_id/resources/:resource_unique_id - Update Resource

Updates a subject resource.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/subjects/subject-uuid-001/resources/resource-uuid-002" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "resource": {
      "name": "Updated Chapter Reference"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "resource-uuid-002",
    "type": "resource",
    "attributes": {
      "unique_id": "resource-uuid-002",
      "name": "Updated Chapter Reference"
    }
  }
}
```

---

### DELETE /subjects/:unique_id/resources/:resource_unique_id - Delete Resource

Deletes a subject resource.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/subjects/subject-uuid-001/resources/resource-uuid-002" \
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

### PUT /subjects/:unique_id/presign_upload - Presign URL

Generates a presigned URL for file upload to a subject.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/subjects/subject-uuid-001/presign_upload" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "filename": "slides.pptx",
      "content_type": "application/vnd.openxmlformats-officedocument.presentationml.presentation"
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
    "presigned_url": "https://storage.example.com/upload?signature=def456",
    "file_url": "https://storage.example.com/subjects/subject-uuid-001/slides.pptx",
    "expires_at": "2025-09-01T11:00:00Z"
  }
}
```

---

### GET /subjects/:unique_id/tests - Get Tests

Gets all content tests for a subject.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subjects/subject-uuid-001/tests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "test-uuid-002",
      "type": "test",
      "attributes": {
        "unique_id": "test-uuid-002",
        "name": "Data Structures Quiz",
        "test_type": "quiz",
        "time_limit": 30,
        "max_score": 50,
        "passing_score": 30,
        "status": "active"
      }
    }
  ]
}
```

---

## Data Models

### Subject
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Subject name |
| `description` | string | Subject description |
| `course_id` | uuid | Parent course ID |
| `order` | integer | Display order within course |
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
// SubjectsService — client.university.subjects
client.university.subjects.list(params?: ListSubjectsParams): Promise<PageResult<Subject>>;
client.university.subjects.get(uniqueId: string): Promise<Subject>;
client.university.subjects.create(data: CreateSubjectRequest): Promise<Subject>;
client.university.subjects.update(uniqueId: string, data: UpdateSubjectRequest): Promise<Subject>;
client.university.subjects.getResources(uniqueId: string): Promise<unknown[]>;
client.university.subjects.getTeacherResources(uniqueId: string, teacherUniqueId: string): Promise<unknown[]>;
client.university.subjects.getTests(uniqueId: string): Promise<unknown[]>;
client.university.subjects.addLesson(uniqueId: string, lessonData: { name: string; description?: string }): Promise<unknown>;
client.university.subjects.addResource(uniqueId: string, resourceData: unknown): Promise<unknown>;
client.university.subjects.updateResource(uniqueId: string, resourceUniqueId: string, resourceData: unknown): Promise<unknown>;
client.university.subjects.deleteResource(uniqueId: string, resourceUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Subject,
  CreateSubjectRequest,
  UpdateSubjectRequest,
  ListSubjectsParams,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.subjects.list();
}
```
