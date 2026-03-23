---
name: 23blocks-university-notes-api
description: Manage 23blocks university notes via REST API. Use for creating and retrieving user notes associated with courses, subjects, or lessons.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Notes API

Complete API reference for 23blocks university note management for courses, subjects, and lessons.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | University API base URL | `https://university.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://university.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://university.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### GET /notes/:unique_id/ - Get Note

Retrieves a specific note by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/notes/note-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "note-uuid-123",
    "type": "note",
    "attributes": {
      "unique_id": "note-uuid-123",
      "user_unique_id": "user-uuid-456",
      "noteable_type": "course",
      "noteable_id": "course-uuid-789",
      "content": "Key takeaways from the introduction module: focus on data structures and algorithm complexity.",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-12T14:15:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Note not found

---

### POST /notes/ - Create or Update Note

Creates a new note or updates an existing one for a course, subject, or lesson.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/notes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "note": {
      "user_unique_id": "user-uuid-456",
      "noteable_type": "course",
      "noteable_id": "course-uuid-789",
      "content": "Key takeaways from the introduction module: focus on data structures and algorithm complexity."
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User unique ID |
| `noteable_type` | enum | Yes | Type of entity: course, subject, lesson |
| `noteable_id` | uuid | Yes | Unique ID of the course, subject, or lesson |
| `content` | text | Yes | Note content |

**Response 201 (Create):**
```json
{
  "data": {
    "id": "note-uuid-new",
    "type": "note",
    "attributes": {
      "unique_id": "note-uuid-new",
      "user_unique_id": "user-uuid-456",
      "noteable_type": "course",
      "noteable_id": "course-uuid-789",
      "content": "Key takeaways from the introduction module: focus on data structures and algorithm complexity.",
      "created_at": "2025-01-12T10:30:00Z",
      "updated_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Response 200 (Update existing):**
```json
{
  "data": {
    "id": "note-uuid-123",
    "type": "note",
    "attributes": {
      "unique_id": "note-uuid-123",
      "user_unique_id": "user-uuid-456",
      "noteable_type": "course",
      "noteable_id": "course-uuid-789",
      "content": "Updated notes with additional observations from week 2.",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-14T09:00:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (e.g., invalid noteable_type)
- `404 Not Found` - Referenced course/subject/lesson not found

---

### Creating Notes for Different Entity Types

#### Course Note
```bash
curl -X POST "$BLOCKS_API_URL/notes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "note": {
      "user_unique_id": "user-uuid-456",
      "noteable_type": "course",
      "noteable_id": "course-uuid-789",
      "content": "Course overview notes..."
    }
  }'
```

#### Subject Note
```bash
curl -X POST "$BLOCKS_API_URL/notes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "note": {
      "user_unique_id": "user-uuid-456",
      "noteable_type": "subject",
      "noteable_id": "subject-uuid-321",
      "content": "Subject-specific study notes..."
    }
  }'
```

#### Lesson Note
```bash
curl -X POST "$BLOCKS_API_URL/notes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "note": {
      "user_unique_id": "user-uuid-456",
      "noteable_type": "lesson",
      "noteable_id": "lesson-uuid-654",
      "content": "Lesson-specific notes and highlights..."
    }
  }'
```

---

## Data Models

### Note
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | Owner user ID |
| `noteable_type` | enum | course, subject, lesson |
| `noteable_id` | uuid | ID of the associated entity |
| `content` | text | Note content |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Noteable Types
| Type | Description |
|------|-------------|
| `course` | Note associated with a course |
| `subject` | Note associated with a subject |
| `lesson` | Note associated with a lesson |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Noteable type must be one of: course, subject, lesson."
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
// NotesService — client.university.notes
client.university.notes.list(params?: ListNotesParams): Promise<PageResult<Note>>;
client.university.notes.get(uniqueId: string): Promise<Note>;
client.university.notes.create(data: CreateNoteRequest): Promise<Note>;
client.university.notes.update(uniqueId: string, data: UpdateNoteRequest): Promise<Note>;
client.university.notes.delete(uniqueId: string): Promise<void>;
client.university.notes.listByAuthor(authorUniqueId: string, params?: ListNotesParams): Promise<PageResult<Note>>;
client.university.notes.listByTarget(targetUniqueId: string, targetType: string, params?: ListNotesParams): Promise<PageResult<Note>>;
client.university.notes.listByCourse(courseUniqueId: string, params?: ListNotesParams): Promise<PageResult<Note>>;
client.university.notes.listByLesson(lessonUniqueId: string, params?: ListNotesParams): Promise<PageResult<Note>>;
client.university.notes.pin(uniqueId: string): Promise<Note>;
client.university.notes.unpin(uniqueId: string): Promise<Note>;
client.university.notes.getReplies(uniqueId: string): Promise<Note[]>;
```

### TypeScript Types

```typescript
import type {
  Note,
  CreateNoteRequest,
  UpdateNoteRequest,
  ListNotesParams,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.notes.list();
}
```
