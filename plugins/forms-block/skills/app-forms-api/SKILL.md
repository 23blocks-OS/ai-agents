---
name: app-forms-api
description: Manage 23blocks App Form instances, schemas, and versions via REST API. Use for dynamic business forms with validation, scoring, and lifecycle management.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# App Forms API

Complete API reference for 23blocks dynamic business forms including instances, schemas, and versions.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://forms.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

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

---

# App Form Instances

## GET /forms/:form_unique_id/instances - List Instances

Lists app form instances with filtering and metadata search.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/forms/9ebed093-574a-4be4-9076-cfca36493330/instances?page=1&records=20&by_status=pending" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `search` | string | No | Search in responses |
| `by_status` | enum | No | Filter by status: pending, in_progress, completed, cancelled |
| `assigned_to` | string | No | Filter by assignee user ID |
| `assigned_by` | string | No | Filter by assigner user ID |
| `metadata_*` | string | No | Search by metadata key (e.g., `metadata_session_id=123`) |

**Metadata Search Examples:**
```bash
# Find by external session ID
GET /forms/{form_id}/instances?metadata_session_id=7688342

# Find by multiple metadata fields
GET /forms/{form_id}/instances?metadata_source=mobile&metadata_campaign=summer2025

# Search with operators
GET /forms/{form_id}/instances?metadata_status[op]=like&metadata_status[value]=pend
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "inst-123",
      "type": "app_form_instance",
      "attributes": {
        "unique_id": "inst-123",
        "status": "pending",
        "assigned_to_unique_id": "user-456",
        "assigned_to_email": "patient@example.com",
        "assigned_to_name": "John Doe",
        "assigned_by_unique_id": "admin-789",
        "assigned_by_email": "admin@example.com",
        "responses": null,
        "calculated_scores": null,
        "metadata": {
          "session_id": "7688342",
          "source": "mobile"
        },
        "token": "abc123xyz",
        "expires_at": "2025-01-20T10:30:00Z",
        "started_at": null,
        "completed_at": null,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 42
  }
}
```

---

## GET /forms/:form_unique_id/instances/:unique_id - Get Instance

Retrieves a specific app form instance with its schema version.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/forms/9ebed093-574a-4be4-9076-cfca36493330/instances/inst-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "inst-123",
    "type": "app_form_instance",
    "attributes": {
      "unique_id": "inst-123",
      "status": "completed",
      "assigned_to_email": "patient@example.com",
      "assigned_to_name": "John Doe",
      "responses": [null, 2, 1, 3, 2, 1, 2, 3],
      "calculated_scores": {
        "total_score": 14,
        "severity": "Moderate anxiety"
      },
      "validation_errors": null,
      "metadata": { "session_id": "7688342" },
      "started_at": "2025-01-10T11:00:00Z",
      "completed_at": "2025-01-10T11:15:00Z"
    },
    "relationships": {
      "form": { "data": { "id": "form-id", "type": "form" } },
      "form_schema_version": { "data": { "id": "version-id", "type": "form_schema_version" } }
    }
  },
  "included": [
    {
      "id": "version-id",
      "type": "form_schema_version",
      "attributes": {
        "version_number": 1,
        "status": "published",
        "form_fields": [
          { "name": "intro", "type": "display", "content": "Over the last 2 weeks..." },
          { "name": "q1", "type": "radio", "label": "Feeling nervous", "options": [...], "scores": [0,1,2,3] }
        ],
        "validation_rules": { "required_fields": ["q1", "q2", "q3"] },
        "scoring_rules": { "total_score": { "type": "sum", "fields": ["q1", "q2", "q3"] } }
      }
    }
  ]
}
```

**Errors:**
- `404 705` - Form not found or invalid form type
- `404 716` - Instance not found

---

## POST /forms/:form_unique_id/instances - Create Instance

Creates and assigns a new app form instance. Automatically sends magic link email.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/forms/9ebed093-574a-4be4-9076-cfca36493330/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "app_form_instance": {
      "assigned_to_email": "patient@example.com",
      "assigned_to_name": "John Doe",
      "assigned_to_unique_id": "external-user-id",
      "assigned_by_name": "Dr. Smith",
      "expires_at": "2025-01-20T23:59:59Z",
      "metadata": {
        "session_id": "7688342",
        "appointment_id": "apt-456",
        "source": "clinic_portal"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `assigned_to_email` | string | Yes | Recipient email (magic link sent here) |
| `assigned_to_name` | string | No | Recipient display name |
| `assigned_to_unique_id` | string | No | External user ID |
| `assigned_by_name` | string | No | Assigner display name |
| `expires_at` | timestamp | No | Form expiration date |
| `metadata` | object | No | Custom metadata for search/tracking |

**Response 201:**
```json
{
  "data": {
    "id": "new-inst-id",
    "type": "app_form_instance",
    "attributes": {
      "unique_id": "new-inst-id",
      "status": "pending",
      "assigned_to_email": "patient@example.com",
      "token": "magic-link-token",
      "magic_link_sent_at": "2025-01-12T10:30:00Z",
      "magic_link_sent_count": 1
    }
  }
}
```

**Errors:**
- `404 705` - Form not found
- `422 706` - No published schema version
- `422 707` - Creation failed (validation errors)

---

## PUT /forms/:form_unique_id/instances/:unique_id - Update Instance

Updates instance responses and metadata.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/forms/form-id/instances/inst-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "app_form_instance": {
      "responses": [null, 2, 1, 3, 2, 1, 2],
      "metadata": {
        "session_id": "7688342",
        "last_saved_page": 2
      }
    }
  }'
```

**Response 200:** Updated instance

**Errors:**
- `422 708` - Cannot edit (completed, expired, or cancelled)
- `422 709` - Update failed (validation errors)

---

## POST /forms/:form_unique_id/instances/:unique_id/start - Start Instance

Marks the instance as started (status: in_progress).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/forms/form-id/instances/inst-123/start" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "inst-123",
    "type": "app_form_instance",
    "attributes": {
      "status": "in_progress",
      "started_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 710` - Start failed

---

## POST /forms/:form_unique_id/instances/:unique_id/submit - Submit Instance

Validates responses, calculates scores, and marks as completed.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/forms/form-id/instances/inst-123/submit" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "app_form_instance": {
      "responses": [null, 2, 1, 3, 2, 1, 2, 3]
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "inst-123",
    "type": "app_form_instance",
    "attributes": {
      "status": "completed",
      "responses": [null, 2, 1, 3, 2, 1, 2, 3],
      "calculated_scores": {
        "total_score": 14,
        "severity": "Moderate anxiety"
      },
      "completed_at": "2025-01-12T10:45:00Z"
    }
  }
}
```

**Errors:**
- `422 711` - Cannot submit (already completed)
- `422 712` - Validation failed (with validation errors)
- `422 713` - Scoring failed

---

## POST /forms/:form_unique_id/instances/:unique_id/cancel - Cancel Instance

Cancels the instance (prevents further edits).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/forms/form-id/instances/inst-123/cancel" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Cancelled instance

**Errors:**
- `422 715` - Cancel failed

---

## POST /forms/:form_unique_id/instances/:unique_id/resend_magic_link - Resend Magic Link

Resends the magic link email.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/forms/form-id/instances/inst-123/resend_magic_link" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Magic link resent successfully",
  "sent_to": "patient@example.com",
  "sent_count": 2
}
```

**Errors:**
- `422 718` - No email address
- `422 719` - Cannot resend for completed form

---

## DELETE /forms/:form_unique_id/instances/:unique_id - Delete Instance

Soft-deletes the instance.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/forms/form-id/instances/inst-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

# Form Schemas

## GET /forms/:form_unique_id/schemas - List Schemas

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/forms/form-id/schemas" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

---

## POST /forms/:form_unique_id/schemas - Create Schema

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/forms/form-id/schemas" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "form_schema": {
      "name": "GAD-7",
      "description": "Generalized Anxiety Disorder 7-item scale"
    }
  }'
```

---

# Form Schema Versions

## GET /forms/:form_id/schemas/:schema_id/versions - List Versions

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/forms/form-id/schemas/schema-id/versions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

---

## POST /forms/:form_id/schemas/:schema_id/versions - Create Version

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/forms/form-id/schemas/schema-id/versions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "form_schema_version": {
      "form_fields": [
        {
          "name": "intro",
          "type": "display",
          "content": "Over the last 2 weeks, how often have you been bothered by the following problems?"
        },
        {
          "name": "q1",
          "type": "radio",
          "label": "Feeling nervous, anxious, or on edge",
          "required": true,
          "options": ["Not at all", "Several days", "More than half the days", "Nearly every day"],
          "scores": [0, 1, 2, 3]
        }
      ],
      "validation_rules": {
        "required_fields": ["q1", "q2", "q3", "q4", "q5", "q6", "q7"]
      },
      "scoring_rules": {
        "total_score": {
          "type": "sum",
          "fields": ["q1", "q2", "q3", "q4", "q5", "q6", "q7"]
        },
        "severity": {
          "type": "range",
          "source": "total_score",
          "ranges": [
            { "min": 0, "max": 4, "label": "Minimal anxiety" },
            { "min": 5, "max": 9, "label": "Mild anxiety" },
            { "min": 10, "max": 14, "label": "Moderate anxiety" },
            { "min": 15, "max": 21, "label": "Severe anxiety" }
          ]
        }
      }
    }
  }'
```

---

## POST /forms/:form_id/schemas/:schema_id/versions/:version_id/publish - Publish Version

Publishes a version (archives previously published versions).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/forms/form-id/schemas/schema-id/versions/version-id/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "version-id",
    "type": "form_schema_version",
    "attributes": {
      "status": "published",
      "published_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

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
               |              |          |          +-- Text response
               |              |          +-- Option index (1 = "Several days")
               |              +-- Option index (2 = "More than half")
               +-- Display field (always null)
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
