---
name: 23blocks-university-courses-api
description: Manage 23blocks University courses via REST API. Use when creating, updating courses, managing enrollment, teachers, groups, resources, assignments, tests, and placement tests.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Courses API

Complete API reference for 23blocks University course management with enrollment, resources, assignments, and assessments.

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

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.courses.list();
}
```
