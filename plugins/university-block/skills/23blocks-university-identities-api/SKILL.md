---
name: 23blocks-university-identities-api
description: Manage 23blocks University student identities via REST API. Use when registering students, managing user profiles, handling enrollment, archiving/restoring users, or retrieving enrolled courses, groups, content trees, and notes.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks University student identity management with enrollment, archiving, and content tree navigation.

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
| GET | `/users/` | List active students |
| GET | `/users/status/archive` | List archived students |
| GET | `/users/:unique_id/` | Get student details |
| POST | `/users/:unique_id/register/` | Register new user |
| PUT | `/users/:unique_id/` | Update user profile |
| PUT | `/users/:unique_id/courses` | Update course enrollment |
| PUT | `/users/:unique_id/restore` | Restore archived user |
| DELETE | `/users/:unique_id/archive` | Archive user |
| GET | `/users/:unique_id/courses` | Get enrolled courses |
| GET | `/users/:unique_id/availables` | Get available courses |
| GET | `/users/:unique_id/groups` | Get user's course groups |
| GET | `/users/:unique_id/content_tree/:course_group_unique_id` | Get content tree |
| GET | `/users/:unique_id/subjects/:subject_unique_id/notes` | Get student notes for subject |

---

## Data Models

### User (Student)
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `email` | string | Email address |
| `phone` | string | Phone number |
| `status` | string | Status: `active`, `archived` |
| `enrollment_date` | timestamp | Date of enrollment |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "User Not Found",
    "detail": "The requested user could not be found."
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
// StudentsService — client.university.students
client.university.students.list(params?: ListStudentsParams): Promise<PageResult<Student>>;
client.university.students.listArchived(params?: ListStudentsParams): Promise<PageResult<Student>>;
client.university.students.get(uniqueId: string): Promise<Student>;
client.university.students.register(uniqueId: string, data: RegisterStudentRequest): Promise<Student>;
client.university.students.update(uniqueId: string, data: UpdateStudentRequest): Promise<Student>;
client.university.students.archive(uniqueId: string): Promise<void>;
client.university.students.restore(uniqueId: string): Promise<Student>;
client.university.students.getCourses(uniqueId: string): Promise<Course[]>;
client.university.students.getAvailableCourses(uniqueId: string): Promise<Course[]>;
client.university.students.getGroups(uniqueId: string): Promise<CourseGroup[]>;
client.university.students.getContentTree(uniqueId: string, courseGroupUniqueId: string): Promise<unknown>;
client.university.students.getAvailability(uniqueId: string): Promise<StudentAvailability[]>;
client.university.students.addAvailability(uniqueId: string, data: { dayOfWeek: number; startTime: string; endTime: string; timezone?: string }): Promise<StudentAvailability>;
client.university.students.updateAvailability(uniqueId: string, availabilityUniqueId: string, data: { dayOfWeek?: number; startTime?: string; endTime?: string; timezone?: string }): Promise<StudentAvailability>;
client.university.students.updateAvailabilitySlots(uniqueId: string, slots: { dayOfWeek: number; startTime: string; endTime: string }[]): Promise<StudentAvailability[]>;
client.university.students.deleteAvailability(uniqueId: string, availabilityUniqueId: string): Promise<void>;
client.university.students.deleteAllAvailability(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Student,
  ListStudentsParams,
  RegisterStudentRequest,
  UpdateStudentRequest,
  StudentAvailability,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.students.list();
}
```
