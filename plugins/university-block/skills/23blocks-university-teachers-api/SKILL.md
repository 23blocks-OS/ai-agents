---
name: 23blocks-university-teachers-api
description: Manage 23blocks University teachers via REST API. Use when listing teachers, managing coaching matches and sessions, setting availability, tracking attendance, handling tests, or promoting students.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Teachers API

Complete API reference for 23blocks University teacher management with coaching, availability, attendance, tests, and student promotion.

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
| GET | `/teachers/` | List active teachers |
| GET | `/teachers/status/archive` | List archived teachers |
| GET | `/teachers/:unique_id` | Get teacher by ID |
| GET | `/teachers/:unique_id/courses` | List teacher's courses |
| GET | `/teachers/:unique_id/groups` | List teacher's course groups |
| GET | `/teachers/:unique_id/content_tree/:course_group_unique_id` | Get content tree for course group |
| GET | `/teachers/:unique_id/users/:user_unique_id/content_tree/:course_group_unique_id` | Get student content tree |
| GET | `/teachers/:unique_id/coaching/active` | List active coaching matches |
| GET | `/teachers/:unique_id/coaching/matches` | List all coaching matches |
| GET | `/teachers/:unique_id/coaching/available` | List students available for coaching |
| POST | `/teachers/:unique_id/coachees/find` | Find coachees matching criteria |
| GET | `/teachers/:unique_id/availability` | Get availability schedule |
| POST | `/teachers/:unique_id/availability` | Add availability slot |
| PUT | `/teachers/:unique_id/availability/:availability_unique_id` | Update availability slot |
| DELETE | `/teachers/:unique_id/availability/:availability_unique_id` | Delete availability slot |
| DELETE | `/teachers/:unique_id/availability` | Delete all availability slots |
| GET | `/teachers/:unique_id/coaching_sessions` | List coaching sessions |
| GET | `/teachers/:unique_id/tests` | List available tests |
| POST | `/teachers/:unique_id/test/:test_unique_id` | Start a test |
| PUT | `/teachers/:unique_id/test/:test_instance_unique_id` | Submit test response |
| PUT | `/teachers/:unique_id/test/:test_instance_unique_id/finish` | Finish test |
| GET | `/teachers/:unique_id/attendance` | Get attendance records |
| POST | `/teachers/:unique_id/attendance` | Register attendance |
| PUT | `/teachers/:unique_id/users/:user_unique_id/promote` | Promote student |

---

## Data Models

### Teacher
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `email` | string | Email address |
| `specialization` | string | Teaching specialization |
| `status` | string | Status: `active`, `archived` |
| `created_at` | timestamp | Creation time |

### Availability
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `teacher_unique_id` | uuid | Teacher ID |
| `day_of_week` | string | Day of week |
| `start_time` | string | Start time (HH:MM) |
| `end_time` | string | End time (HH:MM) |

### CoachingMatch
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `teacher_unique_id` | uuid | Teacher/coach ID |
| `student_unique_id` | uuid | Student/coachee ID |
| `status` | string | Status: `active`, `completed`, `cancelled` |
| `matched_at` | timestamp | Match time |

### CoachingSession
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `teacher_unique_id` | uuid | Teacher ID |
| `student_unique_id` | uuid | Student ID |
| `scheduled_at` | timestamp | Scheduled time |
| `duration_minutes` | integer | Session duration |
| `status` | string | Status: `scheduled`, `completed`, `cancelled` |
| `notes` | string | Session notes |

### Attendance
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `teacher_unique_id` | uuid | Recording teacher ID |
| `user_unique_id` | uuid | Student ID |
| `course_group_unique_id` | uuid | Course group ID |
| `date` | date | Attendance date |
| `status` | string | Status: `present`, `absent`, `late`, `excused` |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Teacher Not Found",
    "detail": "The requested teacher could not be found."
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
// TeachersService — client.university.teachers
client.university.teachers.list(params?: ListTeachersParams): Promise<PageResult<Teacher>>;
client.university.teachers.listArchived(params?: ListTeachersParams): Promise<PageResult<Teacher>>;
client.university.teachers.get(uniqueId: string): Promise<Teacher>;
client.university.teachers.getCourses(uniqueId: string): Promise<Course[]>;
client.university.teachers.getGroups(uniqueId: string): Promise<CourseGroup[]>;
client.university.teachers.getAvailability(uniqueId: string): Promise<TeacherAvailability[]>;
client.university.teachers.addAvailability(uniqueId: string, data: CreateTeacherAvailabilityRequest): Promise<TeacherAvailability>;
client.university.teachers.updateAvailability(uniqueId: string, availabilityUniqueId: string, data: UpdateTeacherAvailabilityRequest): Promise<TeacherAvailability>;
client.university.teachers.deleteAvailability(uniqueId: string, availabilityUniqueId: string): Promise<void>;
client.university.teachers.deleteAllAvailability(uniqueId: string): Promise<void>;
client.university.teachers.getContentTree(uniqueId: string, courseGroupUniqueId: string): Promise<unknown>;
client.university.teachers.getStudentContentTree(uniqueId: string, userUniqueId: string, courseGroupUniqueId: string): Promise<unknown>;
client.university.teachers.promoteStudent(uniqueId: string, userUniqueId: string): Promise<unknown>;
```

### TypeScript Types

```typescript
import type {
  Teacher,
  ListTeachersParams,
  TeacherAvailability,
  CreateTeacherAvailabilityRequest,
  UpdateTeacherAvailabilityRequest,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.teachers.list();
}
```
