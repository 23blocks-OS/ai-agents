---
name: 23blocks-university-attendance-api
description: Manage 23blocks university attendance and availability via REST API. Use for tracking student and teacher attendance, managing availability slots, and bulk updating schedules.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Attendance API

Complete API reference for 23blocks university attendance tracking and availability slot management.

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

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Attendance record already exists for this date."
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

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.attendance.list();
}
```
