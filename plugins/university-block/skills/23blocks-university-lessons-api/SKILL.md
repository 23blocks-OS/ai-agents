---
name: 23blocks-university-lessons-api
description: "University Block lessons: CRUD, completion, resources, tests. Use for lesson content."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Lessons API

Complete API reference for 23blocks University lesson management with resources, completion tracking, and tests.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://university.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/lessons/` | List lessons |
| GET | `/lessons/:unique_id/` | Get lesson with resources |
| POST | `/lessons/` | Create lesson |
| PUT | `/lessons/:unique_id` | Update lesson |
| PUT | `/lessons/:unique_id/completed` | Mark lesson completed |
| GET | `/lessons/:unique_id/resources` | Get lesson resources |
| POST | `/lessons/:unique_id/resources` | Add resource |
| PUT | `/lessons/:unique_id/resources/:resource_unique_id` | Update resource |
| DELETE | `/lessons/:unique_id/resources/:resource_unique_id` | Delete resource |
| PUT | `/lessons/:unique_id/presign_upload` | Presign upload URL |
| GET | `/lessons/:unique_id/tests` | Get lesson tests |

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Lesson Not Found","detail":"The requested lesson could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-university`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useUniversityBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// LessonsService — client.university.lessons
client.university.lessons.list(params?: ListLessonsParams): Promise<PageResult<Lesson>>;
client.university.lessons.get(uniqueId: string): Promise<Lesson>;
client.university.lessons.create(data: CreateLessonRequest): Promise<Lesson>;
client.university.lessons.update(uniqueId: string, data: UpdateLessonRequest): Promise<Lesson>;
client.university.lessons.delete(uniqueId: string): Promise<void>;
client.university.lessons.reorder(courseUniqueId: string, data: ReorderLessonsRequest): Promise<Lesson[]>;
client.university.lessons.listByCourse(courseUniqueId: string, params?: ListLessonsParams): Promise<PageResult<Lesson>>;
```

### TypeScript Types

```typescript
import type {
  Lesson,
  CreateLessonRequest,
  UpdateLessonRequest,
  ListLessonsParams,
  ReorderLessonsRequest,
  LessonContentType,
} from '@23blocks/block-university';
```
