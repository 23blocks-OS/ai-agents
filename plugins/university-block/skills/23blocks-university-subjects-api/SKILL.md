---
name: 23blocks-university-subjects-api
description: Manage 23blocks University subjects via REST API. Use when creating, updating subjects, adding lessons, managing resources, or retrieving subject tests.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Subjects API

Complete API reference for 23blocks University subject management with lessons, resources, and tests.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | University API base URL | `https://university.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/subjects/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

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

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.subjects.list();
}
```
