---
name: 23blocks-university-coaching-api
description: Manage 23blocks university coaching matches and sessions via REST API. Use for creating student-teacher pairings, scheduling coaching sessions, tracking check-ins/check-outs, and managing session notes.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Coaching API

Complete API reference for 23blocks university coaching match management and session scheduling.

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
| POST | `/matches/` | Create coaching match |
| PUT | `/matches/:unique_id/activate` | Activate match |
| PUT | `/matches/:unique_id/deactivate` | Deactivate match |
| DELETE | `/matches/:unique_id` | Delete match |
| POST | `/matches/evaluate` | Evaluate potential matches |
| POST | `/matches/availabilities` | Evaluate availability compatibility |
| GET | `/users/:unique_id/coaching/active` | Get student's active match |
| GET | `/users/:unique_id/coaching/matches` | List student's all matches |
| GET | `/users/:unique_id/coaching/available` | List available coaches for student |
| POST | `/users/:unique_id/coaches/find` | Find coaches matching criteria |
| GET | `/coaching/sessions/` | List coaching sessions |
| GET | `/coaching/sessions/:unique_id` | Get coaching session |
| POST | `/coaching/sessions` | Create coaching session |
| PUT | `/coaching/sessions/:unique_id` | Update coaching session |
| DELETE | `/coaching/sessions/:unique_id` | Cancel coaching session |
| PUT | `/coaching/sessions/:unique_id/students/confirmation` | Student confirms session |
| PUT | `/coaching/sessions/:unique_id/students/checking` | Student check-in |
| PUT | `/coaching/sessions/:unique_id/students/checkout` | Student check-out |
| PUT | `/coaching/sessions/:unique_id/students/notes` | Update student notes |
| PUT | `/coaching/sessions/:unique_id/teachers/confirmation` | Teacher confirms session |
| PUT | `/coaching/sessions/:unique_id/teachers/checking` | Teacher check-in |
| PUT | `/coaching/sessions/:unique_id/teachers/checkout` | Teacher check-out |
| PUT | `/coaching/sessions/:unique_id/admin/notes` | Update admin notes |
| GET | `/users/:unique_id/coaching_sessions` | List student's sessions |
| GET | `/teachers/:unique_id/coaching_sessions` | List teacher's sessions |

---

## Data Models

### Match
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `student_id` | uuid | Student unique ID |
| `teacher_id` | uuid | Teacher/coach unique ID |
| `status` | enum | pending, active, inactive |
| `created_at` | timestamp | Creation time |

### CoachingSession
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `match_id` | uuid | Associated match ID |
| `scheduled_at` | datetime | Scheduled date/time |
| `duration` | integer | Duration in minutes |
| `location` | string | Session location |
| `student_status` | enum | pending, confirmed, checked_in, checked_out |
| `teacher_status` | enum | pending, confirmed, checked_in, checked_out |
| `notes` | text | Session notes |
| `admin_notes` | text | Admin notes |
| `status` | enum | scheduled, confirmed, in_progress, completed, cancelled |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Match already exists for this student-teacher pair."
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
// CoachingSessionsService — client.university.coachingSessions
client.university.coachingSessions.list(params?: ListCoachingSessionsParams): Promise<PageResult<CoachingSession>>;
client.university.coachingSessions.create(data: CreateCoachingSessionRequest): Promise<CoachingSession>;
client.university.coachingSessions.update(uniqueId: string, data: UpdateCoachingSessionRequest): Promise<CoachingSession>;
client.university.coachingSessions.delete(uniqueId: string): Promise<void>;
client.university.coachingSessions.getByStudent(studentUniqueId: string): Promise<CoachingSession[]>;
client.university.coachingSessions.getByTeacher(teacherUniqueId: string): Promise<CoachingSession[]>;
client.university.coachingSessions.studentConfirm(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.studentCheckIn(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.studentCheckOut(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.studentNotes(uniqueId: string, notes: string): Promise<CoachingSession>;
client.university.coachingSessions.teacherConfirm(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.teacherCheckIn(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.teacherCheckOut(uniqueId: string): Promise<CoachingSession>;
client.university.coachingSessions.adminNotes(uniqueId: string, notes: string): Promise<CoachingSession>;
```

### TypeScript Types

```typescript
import type {
  CoachingSession,
  CreateCoachingSessionRequest,
  UpdateCoachingSessionRequest,
  ListCoachingSessionsParams,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.coachingSessions.list();
}
```
