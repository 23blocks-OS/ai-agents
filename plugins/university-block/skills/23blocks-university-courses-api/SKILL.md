---
name: 23blocks-university-courses-api
description: "University Block courses: enrollment, teachers, groups, resources, assignments, tests, placement. Use for course setup."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Courses API

Complete API reference for 23blocks University course management with enrollment, resources, assignments, and assessments.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://university.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/courses/` | List courses |
| GET | `/courses/:unique_id/` | Get course with subjects and resources |
| POST | `/courses/` | Create course |
| PUT | `/courses/:unique_id` | Update course |
| GET | `/courses/:unique_id/teachers` | List course teachers |
| GET | `/courses/:unique_id/students` | List course students |
| POST | `/courses/:unique_id/enrollment` | Enroll student |
| POST | `/courses/:unique_id/teacher` | Add teacher to course |
| PUT | `/courses/:unique_id/enrollment/:enrollment_code` | Validate enrollment |
| GET | `/courses/:unique_id/groups` | List course groups |
| POST | `/course/:unique_id/course_groups/enrollment` | Register student to group |
| GET | `/courses/:unique_id/resources` | Get course resources |
| POST | `/courses/:unique_id/resources` | Add resource |
| PUT | `/courses/:unique_id/resources/:resource_unique_id` | Update resource |
| DELETE | `/courses/:unique_id/resources/:resource_unique_id` | Delete resource |
| PUT | `/courses/:unique_id/presign_upload` | Presign upload URL |
| GET | `/courses/:unique_id/assignments/` | List course assignments |
| POST | `/courses/:unique_id/assignments` | Create assignment |
| GET | `/courses/:unique_id/tests` | List course tests |
| GET | `/courses/:unique_id/placement` | List placement tests |
| POST | `/courses/:unique_id/placement/` | Create placement test |

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Failed","detail":"Name can't be blank."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-university`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useUniversityBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// CoursesService — client.university.courses
client.university.courses.list(params?: ListCoursesParams): Promise<PageResult<Course>>;
client.university.courses.get(uniqueId: string): Promise<Course>;
client.university.courses.create(data: CreateCourseRequest): Promise<Course>;
client.university.courses.update(uniqueId: string, data: UpdateCourseRequest): Promise<Course>;
client.university.courses.delete(uniqueId: string): Promise<void>;
client.university.courses.publish(uniqueId: string): Promise<Course>;
client.university.courses.unpublish(uniqueId: string): Promise<Course>;
client.university.courses.listByInstructor(instructorUniqueId: string, params?: ListCoursesParams): Promise<PageResult<Course>>;
client.university.courses.listByCategory(categoryUniqueId: string, params?: ListCoursesParams): Promise<PageResult<Course>>;
```

### TypeScript Types

```typescript
import type {
  Course,
  CreateCourseRequest,
  UpdateCourseRequest,
  ListCoursesParams,
} from '@23blocks/block-university';
```
