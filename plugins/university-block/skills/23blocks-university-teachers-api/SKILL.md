---
name: 23blocks-university-teachers-api
description: "University Block teacher view: courses, groups, coaching, availability, attendance, tests, promotions. Use for teacher-side work."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Teachers API

Complete API reference for 23blocks University teacher management with coaching, availability, attendance, tests, and student promotion.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://university.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Teacher Not Found","detail":"The requested teacher could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-university`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useUniversityBlock()` from `@23blocks/react`.

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
