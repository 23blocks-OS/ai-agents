---
name: 23blocks-university-notes-api
description: "University Block user notes on courses, subjects or lessons. Use for study notes."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Notes API

Complete API reference for 23blocks university note management for courses, subjects, and lessons.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://university.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /notes/:unique_id/ - Get Note

Retrieves a specific note by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/notes/note-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Error","detail":"Noteable type must be one of: course, subject, lesson."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-university`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useUniversityBlock()` from `@23blocks/react`.

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
