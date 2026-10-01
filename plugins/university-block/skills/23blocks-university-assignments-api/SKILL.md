---
name: 23blocks-university-assignments-api
description: "University Block assignments: CRUD, submissions, responses, grading. Use for homework."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Assignments API

Complete API reference for 23blocks University assignment management with response submission and grading workflows.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://university.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> **Note:** There is no standalone `GET /assignments/:unique_id` endpoint. Assignments are accessed via their parent course: `GET /courses/:unique_id/assignments/`.

### POST /assignments/ - Create Assignment

Creates a new assignment.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assignments/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
    "totalPages": 2,
    "totalRecords": 25,
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Submission Failed","detail":"Assignment due date has passed."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-university`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useUniversityBlock()` from `@23blocks/react`.

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
