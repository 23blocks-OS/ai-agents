---
name: 23blocks-university-subjects-api
description: "University Block subjects: CRUD, lessons, resources, tests. Use for course subjects."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Subjects API

Complete API reference for 23blocks University subject management with lessons, resources, and tests.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://university.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/subjects/` | List subjects |
| GET | `/subjects/:unique_id` | Get subject with lessons |
| POST | `/subjects/` | Create subject |
| PUT | `/subjects/:unique_id` | Update subject |
| POST | `/subjects/:unique_id/lessons` | Add lesson to subject |
| GET | `/subjects/:unique_id/resources` | Get subject resources |
| POST | `/subjects/:unique_id/resources` | Add resource |
| PUT | `/subjects/:unique_id/resources/:resource_unique_id` | Update resource |
| DELETE | `/subjects/:unique_id/resources/:resource_unique_id` | Delete resource |
| PUT | `/subjects/:unique_id/presign_upload` | Presign upload URL |
| GET | `/subjects/:unique_id/tests` | Get subject tests |

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Failed","detail":"Name can't be blank."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-university`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useUniversityBlock()` from `@23blocks/react`.

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
