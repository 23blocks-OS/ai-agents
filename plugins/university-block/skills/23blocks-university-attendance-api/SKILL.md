---
name: 23blocks-university-attendance-api
description: "University Block attendance and availability slots for students and teachers. Use for scheduling and attendance."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Attendance API

Complete API reference for 23blocks university attendance tracking and availability slot management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://university.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users/:unique_id/attendance` | List student attendance |
| POST | `/users/:unique_id/attendance` | Register student attendance |
| GET | `/teachers/:unique_id/attendance` | List teacher attendance |
| POST | `/teachers/:unique_id/attendance` | Register teacher attendance |
| GET | `/users/:unique_id/availability` | List student availability slots |
| POST | `/users/:unique_id/availability` | Add student availability slot |
| PUT | `/users/:unique_id/availability/:availability_unique_id` | Update availability slot |
| PUT | `/users/:unique_id/availabilities/slots` | Bulk update availability slots |
| DELETE | `/users/:unique_id/availability/:availability_unique_id` | Delete availability slot |
| DELETE | `/users/:unique_id/availability` | Delete all availability slots |

---

## Data Models

### Attendance
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | User unique ID |
| `user_type` | enum | student, teacher |
| `date` | date | Attendance date |
| `status` | enum | present, absent, late, excused |
| `notes` | string | Additional notes |
| `created_at` | timestamp | Creation time |

### Availability
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | User unique ID |
| `day_of_week` | string | monday, tuesday, wednesday, thursday, friday, saturday, sunday |
| `start_time` | string | Start time (HH:MM) |
| `end_time` | string | End time (HH:MM) |
| `recurrence` | string | weekly, biweekly |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Error","detail":"Attendance record already exists for this date."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-university`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useUniversityBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// AttendanceService — client.university.attendance
client.university.attendance.list(params?: ListAttendanceParams): Promise<PageResult<Attendance>>;
client.university.attendance.get(uniqueId: string): Promise<Attendance>;
client.university.attendance.create(data: CreateAttendanceRequest): Promise<Attendance>;
client.university.attendance.update(uniqueId: string, data: UpdateAttendanceRequest): Promise<Attendance>;
client.university.attendance.delete(uniqueId: string): Promise<void>;
client.university.attendance.bulkCreate(data: BulkAttendanceRequest): Promise<Attendance[]>;
client.university.attendance.listByLesson(lessonUniqueId: string, params?: ListAttendanceParams): Promise<PageResult<Attendance>>;
client.university.attendance.listByStudent(studentUniqueId: string, params?: ListAttendanceParams): Promise<PageResult<Attendance>>;
client.university.attendance.listByCourse(courseUniqueId: string, params?: ListAttendanceParams): Promise<PageResult<Attendance>>;
client.university.attendance.getStudentStats(studentUniqueId: string, courseUniqueId?: string): Promise<AttendanceStats>;
client.university.attendance.verify(uniqueId: string): Promise<Attendance>;
```

### TypeScript Types

```typescript
import type {
  Attendance,
  CreateAttendanceRequest,
  UpdateAttendanceRequest,
  ListAttendanceParams,
  BulkAttendanceRequest,
  AttendanceStats,
} from '@23blocks/block-university';
```
