---
name: 23blocks-forms-app-forms-api
description: Manage 23blocks App Form instances, schemas, and versions via REST API. Use for dynamic business forms with validation, scoring, and lifecycle management.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# App Forms API

Complete API reference for 23blocks dynamic business forms including instances, schemas, and versions.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/forms" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/forms/:form_id/instances` | List instances (filter by status, assignee, metadata) |
| GET | `/forms/:form_id/instances/:id` | Get instance with schema version |
| POST | `/forms/:form_id/instances` | Create + assign instance (sends magic link) |
| PUT | `/forms/:form_id/instances/:id` | Update responses and metadata |
| DELETE | `/forms/:form_id/instances/:id` | Soft-delete instance |
| POST | `/forms/:form_id/instances/:id/start` | Start instance (pending → in_progress) |
| POST | `/forms/:form_id/instances/:id/submit` | Submit, validate, score, complete |
| POST | `/forms/:form_id/instances/:id/cancel` | Cancel instance |
| POST | `/forms/:form_id/instances/:id/resend_magic_link` | Resend magic link email |
| GET | `/forms/:form_id/schemas` | List form schemas |
| POST | `/forms/:form_id/schemas` | Create form schema |
| GET | `/forms/:form_id/schemas/:schema_id/versions` | List schema versions |
| POST | `/forms/:form_id/schemas/:schema_id/versions` | Create draft version |
| POST | `/forms/:form_id/schemas/:schema_id/versions/:version_id/publish` | Publish version |

---

## Data Models

### App Form Instance
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `status` | enum | pending, in_progress, completed, cancelled |
| `assigned_to_unique_id` | string | Assignee external ID |
| `assigned_to_email` | string | Assignee email |
| `assigned_to_name` | string | Assignee display name |
| `assigned_by_unique_id` | uuid | Assigner user ID |
| `assigned_by_email` | string | Assigner email |
| `responses` | array | Array of responses (by field index) |
| `calculated_scores` | object | Computed scores |
| `validation_errors` | object | Validation errors |
| `metadata` | object | Custom metadata |
| `token` | string | Magic link token |
| `expires_at` | timestamp | Expiration time |
| `started_at` | timestamp | When user started |
| `completed_at` | timestamp | Completion time |

### Form Schema Version
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `version_number` | integer | Sequential version |
| `status` | enum | draft, published, archived |
| `form_fields` | array | Field definitions |
| `validation_rules` | object | Validation configuration |
| `scoring_rules` | object | Scoring configuration |
| `published_at` | timestamp | Publication time |

### Response Array Format
```
Schema fields: [intro(display), q1(radio), q2(radio), q3(text)]
Responses:     [null,          2,         1,         "notes"]
               ^              ^          ^          ^
               display(null)  option idx option idx text value
```

### Validation Rules
| Rule | Description |
|------|-------------|
| `required_fields` | Array of required field names |
| `min_length` | Minimum string length |
| `max_length` | Maximum string length |
| `pattern` | Regex pattern |
| `range` | Numeric range (min/max) |
| `custom` | Custom validator name |

### Scoring Rules
| Type | Description |
|------|-------------|
| `sum` | Add up field scores |
| `average` | Calculate average |
| `count` | Count occurrences |
| `range` | Map score to label |
| `conditional` | If/then logic |
| `custom` | Custom calculator |

---

## Error Response Format

```json
{ "errors": [{ "status": "422", "code": "validation_error", "title": "Validation Failed", "detail": "..." }] }
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-forms
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
// FormInstancesService — client.forms.formInstances
list(formUniqueId: string, params?: ListFormInstancesParams): Promise<PageResult<FormInstance>>;
get(formUniqueId: string, uniqueId: string): Promise<FormInstance>;
create(formUniqueId: string, data: CreateFormInstanceRequest): Promise<FormInstance>;
update(formUniqueId: string, uniqueId: string, data: UpdateFormInstanceRequest): Promise<FormInstance>;
delete(formUniqueId: string, uniqueId: string): Promise<void>;
start(formUniqueId: string, uniqueId: string): Promise<FormInstance>;
submit(formUniqueId: string, uniqueId: string): Promise<FormInstance>;
cancel(formUniqueId: string, uniqueId: string): Promise<FormInstance>;
resendMagicLink(formUniqueId: string, uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  FormInstance,
  CreateFormInstanceRequest,
  UpdateFormInstanceRequest,
  SubmitFormInstanceRequest,
  ListFormInstancesParams,
} from '@23blocks/block-forms';
```

### React Hook

```typescript
import { useFormsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFormsBlock();

  // Example: list instances for a form
  const result = await client.forms.formInstances.list('form-unique-id');
}
```
