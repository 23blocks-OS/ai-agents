---
name: surveys-api
description: Manage 23blocks survey instances via REST API. Use for feedback collection, NPS surveys, and data gathering with magic link access.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Surveys API

Complete API reference for 23blocks survey instance management.

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
curl -X GET "$BLOCKS_API_URL/surveys/{form_id}/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /surveys/:form_unique_id/instances - List Survey Instances

Lists survey instances for a specific survey form.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/surveys/form-123/instances?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `search` | string | No | Search text |
| `sort` | string | No | Sort field (prefix `-` for desc) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "survey-inst-123",
      "type": "survey_instance",
      "attributes": {
        "unique_id": "survey-inst-123",
        "user_unique_id": "user-456",
        "assigned_to_email": "customer@example.com",
        "assigned_to_name": "Jane Smith",
        "status": "pending",
        "response": null,
        "token": "magic-link-token",
        "first_accessed_at": null,
        "completed_at": null,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### GET /surveys/:form_unique_id/instances/status/:status - List by Status

Lists survey instances filtered by status.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/surveys/form-123/instances/status/completed" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Status Values:**
- `pending` - Created but not accessed
- `in_progress` - Accessed but not completed
- `completed` - Submitted
- `expired` - Token expired

---

### GET /surveys/:form_unique_id/instances/:unique_id - Get Survey Instance

Retrieves a specific survey instance.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/surveys/form-123/instances/survey-inst-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "survey-inst-123",
    "type": "survey_instance",
    "attributes": {
      "unique_id": "survey-inst-123",
      "user_unique_id": "user-456",
      "assigned_to_email": "customer@example.com",
      "assigned_to_name": "Jane Smith",
      "status": "completed",
      "response": {
        "q1": 5,
        "q2": "Great service!",
        "q3": ["speed", "quality"]
      },
      "token": "magic-link-token",
      "token_expires_at": "2025-01-20T10:30:00Z",
      "token_revoked_at": "2025-01-12T15:00:00Z",
      "first_accessed_at": "2025-01-12T14:30:00Z",
      "completed_at": "2025-01-12T15:00:00Z",
      "otp_verified_at": null,
      "metadata": {
        "order_id": "ORD-789"
      },
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "form": {
        "data": { "id": "form-123", "type": "form" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Survey instance not found

---

### POST /surveys/:form_unique_id/instances - Create Survey Instance

Creates a new survey instance and optionally sends magic link.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/surveys/form-123/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "survey_instance": {
      "user_unique_id": "user-456",
      "assigned_to_email": "customer@example.com",
      "assigned_to_name": "Jane Smith",
      "expires_at": "2025-01-20T23:59:59Z",
      "metadata": {
        "order_id": "ORD-789",
        "product": "Premium Plan"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | string | No | External user ID |
| `assigned_to_email` | string | Yes | Recipient email |
| `assigned_to_name` | string | No | Recipient name |
| `expires_at` | timestamp | No | Token expiration |
| `metadata` | object | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "new-survey-inst",
    "type": "survey_instance",
    "attributes": {
      "unique_id": "new-survey-inst",
      "status": "pending",
      "token": "new-magic-token",
      "magic_link_sent_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /surveys/:form_unique_id/instances/:unique_id - Update Survey Instance

Updates survey instance data.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/surveys/form-123/instances/survey-inst-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "survey_instance": {
      "response": {
        "q1": 5,
        "q2": "Updated feedback"
      },
      "metadata": {
        "order_id": "ORD-789",
        "updated": true
      }
    }
  }'
```

---

### PUT /surveys/:form_unique_id/instances/:unique_id/status - Update Status

Updates the survey instance status.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/surveys/form-123/instances/survey-inst-123/status" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "completed"
  }'
```

**Status Values:**
| Status | Description |
|--------|-------------|
| `pending` | Awaiting access |
| `in_progress` | Started but not finished |
| `completed` | Submitted |
| `expired` | Token expired |

---

### POST /surveys/:form_unique_id/instances/:unique_id/resend_magic_link - Resend Magic Link

Resends the magic link email.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/surveys/form-123/instances/survey-inst-123/resend_magic_link" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Magic link resent successfully",
  "sent_to": "customer@example.com",
  "sent_count": 2
}
```

---

### DELETE /surveys/:form_unique_id/instances/:unique_id - Delete Survey Instance

Soft-deletes a survey instance.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/surveys/form-123/instances/survey-inst-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### POST /surveys/users - List by User

Lists all surveys for a specific user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/surveys/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "user-456"
  }'
```

---

## Data Models

### Survey Instance
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | string | External user ID |
| `assigned_to_email` | string | Recipient email |
| `assigned_to_name` | string | Recipient name |
| `status` | enum | pending, in_progress, completed, expired |
| `response` | object | Survey responses |
| `token` | string | Magic link token |
| `token_expires_at` | timestamp | Token expiration |
| `token_revoked_at` | timestamp | Token revocation time |
| `first_accessed_at` | timestamp | First access time |
| `completed_at` | timestamp | Completion time |
| `otp_verified_at` | timestamp | OTP verification time |
| `metadata` | object | Custom metadata |
| `created_at` | timestamp | Creation time |

---

## Error Codes

| HTTP Status | Code | Description |
|-------------|------|-------------|
| 404 | Not Found | Survey instance not found |
| 422 | Unprocessable Entity | Validation error |
| 400 | Bad Request | Invalid parameters |

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
// SurveysService — client.forms.surveys
list(formUniqueId: string, params?: ListSurveysParams): Promise<PageResult<Survey>>;
listByStatus(formUniqueId: string, status: SurveyStatus, params?: ListSurveysParams): Promise<PageResult<Survey>>;
get(formUniqueId: string, uniqueId: string): Promise<Survey>;
create(formUniqueId: string, data: CreateSurveyRequest): Promise<Survey>;
update(formUniqueId: string, uniqueId: string, data: UpdateSurveyRequest): Promise<Survey>;
delete(formUniqueId: string, uniqueId: string): Promise<void>;
updateStatus(formUniqueId: string, uniqueId: string, data: UpdateSurveyStatusRequest): Promise<Survey>;
resendMagicLink(formUniqueId: string, uniqueId: string): Promise<void>;
listByUser(userUniqueId: string, params?: ListSurveysParams): Promise<PageResult<Survey>>;
```

### TypeScript Types

```typescript
import type {
  Survey,
  SurveyStatus,
  CreateSurveyRequest,
  UpdateSurveyRequest,
  UpdateSurveyStatusRequest,
  ListSurveysParams,
} from '@23blocks/block-forms';
```

### React Hook

```typescript
import { useFormsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFormsBlock();

  // Example: list survey instances for a form
  const result = await client.forms.surveys.list('form-unique-id');
}
```
