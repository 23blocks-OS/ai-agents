---
name: 23blocks-university-lessons-api
description: Manage 23blocks University lessons via REST API. Use when creating, updating lessons, marking completion, managing resources, or retrieving lesson tests.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Lessons API

Complete API reference for 23blocks University lesson management with resources, completion tracking, and tests.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | University API base URL | `https://university.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Lesson Not Found",
    "detail": "The requested lesson could not be found."
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

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.lessons.list();
}
```
