---
name: university-assignments-api
description: Manage 23blocks University assignments via REST API. Use when creating, updating, deleting assignments, managing student/teacher responses, submitting work, or grading submissions.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Assignments API

Complete API reference for 23blocks University assignment management with response submission and grading workflows.

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
curl -X GET "$BLOCKS_API_URL/assignments/assignment-uuid-001/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /assignments/:unique_id/ - Get Assignment

Retrieves a single assignment by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assignments/assignment-uuid-001/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "assignment-uuid-001",
    "type": "assignment",
    "attributes": {
      "unique_id": "assignment-uuid-001",
      "title": "Homework 1: Arrays",
      "description": "Complete exercises on array operations",
      "course_id": "course-uuid-456",
      "due_date": "2025-09-15T23:59:00Z",
      "max_score": 100,
      "assignment_type": "homework",
      "status": "active",
      "created_at": "2025-09-01T10:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Assignment not found

---

### POST /assignments/ - Create Assignment

Creates a new assignment.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assignments/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "assignment": {
      "title": "Homework 1: Arrays",
      "description": "Complete exercises on array operations",
      "course_id": "course-uuid-456",
      "due_date": "2025-09-15T23:59:00Z",
      "max_score": 100,
      "assignment_type": "homework"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | Assignment title |
| `description` | string | No | Assignment description |
| `course_id` | uuid | Yes | Parent course ID |
| `due_date` | timestamp | No | Submission deadline |
| `max_score` | integer | No | Maximum possible score |
| `assignment_type` | string | No | Type: `homework`, `project`, `essay`, `lab` |

**Response 201:**
```json
{
  "data": {
    "id": "assignment-uuid-001",
    "type": "assignment",
    "attributes": {
      "unique_id": "assignment-uuid-001",
      "title": "Homework 1: Arrays",
      "description": "Complete exercises on array operations",
      "course_id": "course-uuid-456",
      "due_date": "2025-09-15T23:59:00Z",
      "max_score": 100,
      "assignment_type": "homework",
      "status": "active",
      "created_at": "2025-09-01T10:00:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /assignment/:unique_id - Update Assignment

Updates an existing assignment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/assignment/assignment-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "assignment": {
      "due_date": "2025-09-20T23:59:00Z",
      "description": "Updated: Complete all exercises including bonus problems"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "assignment-uuid-001",
    "type": "assignment",
    "attributes": {
      "unique_id": "assignment-uuid-001",
      "title": "Homework 1: Arrays",
      "description": "Updated: Complete all exercises including bonus problems",
      "due_date": "2025-09-20T23:59:00Z",
      "status": "active"
    }
  }
}
```

---

### DELETE /assignment/:unique_id - Delete Assignment

Deletes an assignment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/assignment/assignment-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Assignment deleted successfully"
}
```

---

### PUT /assignments/:unique_id/presign_upload - Presign Upload

Generates a presigned URL for file upload to an assignment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/assignments/assignment-uuid-001/presign_upload" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "filename": "assignment-instructions.pdf",
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
    "presigned_url": "https://storage.example.com/upload?signature=jkl012",
    "file_url": "https://storage.example.com/assignments/assignment-uuid-001/assignment-instructions.pdf",
    "expires_at": "2025-09-01T11:00:00Z"
  }
}
```

---

### GET /assignments/:unique_id/responses/teachers/:teacher_unique_id - Teacher Responses

Retrieves all responses for an assignment as viewed by a teacher.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assignments/assignment-uuid-001/responses/teachers/teacher-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "response-uuid-001",
      "type": "assignment_response",
      "attributes": {
        "unique_id": "response-uuid-001",
        "assignment_unique_id": "assignment-uuid-001",
        "user_unique_id": "student-uuid-123",
        "content": "Solution to array exercises",
        "file_url": "https://storage.example.com/submissions/response-001.pdf",
        "score": null,
        "graded": false,
        "submitted_at": "2025-09-14T18:00:00Z"
      }
    }
  ],
  "meta": {
    "total_count": 25,
    "graded_count": 10,
    "pending_count": 15
  }
}
```

---

### GET /assignments/:unique_id/responses/students/:student_unique_id - Student Responses

Retrieves a specific student's responses for an assignment.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assignments/assignment-uuid-001/responses/students/student-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "response-uuid-001",
      "type": "assignment_response",
      "attributes": {
        "unique_id": "response-uuid-001",
        "assignment_unique_id": "assignment-uuid-001",
        "content": "Solution to array exercises",
        "file_url": "https://storage.example.com/submissions/response-001.pdf",
        "score": 85,
        "graded": true,
        "feedback": "Good work, minor issues with time complexity analysis",
        "submitted_at": "2025-09-14T18:00:00Z",
        "graded_at": "2025-09-16T10:00:00Z"
      }
    }
  ]
}
```

---

### POST /assignments/:unique_id/response - Submit Response

Submits a response/solution for an assignment.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assignments/assignment-uuid-001/response" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "response": {
      "content": "My solution to the array exercises",
      "file_url": "https://storage.example.com/submissions/my-solution.pdf"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `content` | string | No | Text content of the response |
| `file_url` | string | No | URL to submitted file |

**Response 201:**
```json
{
  "data": {
    "id": "response-uuid-002",
    "type": "assignment_response",
    "attributes": {
      "unique_id": "response-uuid-002",
      "assignment_unique_id": "assignment-uuid-001",
      "content": "My solution to the array exercises",
      "file_url": "https://storage.example.com/submissions/my-solution.pdf",
      "graded": false,
      "submitted_at": "2025-09-14T18:00:00Z"
    }
  }
}
```

---

### PUT /assignments/:unique_id/response/:response_unique_id/grade - Grade Response

Grades a student's assignment response.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/assignments/assignment-uuid-001/response/response-uuid-001/grade" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "grade": {
      "score": 85,
      "feedback": "Good work, minor issues with time complexity analysis"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `score` | integer | Yes | Numeric score |
| `feedback` | string | No | Teacher feedback text |

**Response 200:**
```json
{
  "data": {
    "id": "response-uuid-001",
    "type": "assignment_response",
    "attributes": {
      "unique_id": "response-uuid-001",
      "score": 85,
      "feedback": "Good work, minor issues with time complexity analysis",
      "graded": true,
      "graded_at": "2025-09-16T10:00:00Z"
    }
  }
}
```

---

## Data Models

### Assignment
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Assignment title |
| `description` | string | Assignment description |
| `course_id` | uuid | Parent course ID |
| `due_date` | timestamp | Submission deadline |
| `max_score` | integer | Maximum possible score |
| `assignment_type` | string | Type: `homework`, `project`, `essay`, `lab` |
| `status` | string | Status: `active`, `inactive`, `draft` |
| `created_at` | timestamp | Creation time |

### AssignmentResponse
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `assignment_unique_id` | uuid | Parent assignment ID |
| `user_unique_id` | uuid | Submitting student ID |
| `content` | string | Text content of response |
| `file_url` | string | URL to submitted file |
| `score` | integer | Graded score |
| `feedback` | string | Teacher feedback |
| `graded` | boolean | Whether response has been graded |
| `submitted_at` | timestamp | Submission time |
| `graded_at` | timestamp | Grading time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Submission Failed",
    "detail": "Assignment due date has passed."
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
// AssignmentsService — client.university.assignments
client.university.assignments.list(params?: ListAssignmentsParams): Promise<PageResult<Assignment>>;
client.university.assignments.get(uniqueId: string): Promise<Assignment>;
client.university.assignments.create(data: CreateAssignmentRequest): Promise<Assignment>;
client.university.assignments.update(uniqueId: string, data: UpdateAssignmentRequest): Promise<Assignment>;
client.university.assignments.delete(uniqueId: string): Promise<void>;
client.university.assignments.listByLesson(lessonUniqueId: string, params?: ListAssignmentsParams): Promise<PageResult<Assignment>>;
```

### TypeScript Types

```typescript
import type {
  Assignment,
  CreateAssignmentRequest,
  UpdateAssignmentRequest,
  ListAssignmentsParams,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.assignments.list();
}
```
