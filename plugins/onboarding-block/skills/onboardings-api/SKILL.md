---
name: onboarding-onboardings-api
description: Manage 23blocks onboarding definitions via REST API. Use when creating onboarding flows, adding or updating steps, deleting steps, viewing onboarding details, or admin-progressing users through onboarding steps.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Onboardings API

Complete API reference for 23blocks onboarding definition management with step configuration and admin user progression.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://onboarding.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Onboarding API base URL | `https://onboarding.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/onboardings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /onboardings - List Onboardings

Lists all onboarding definitions with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/onboardings?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "onboarding-uuid-456",
      "type": "onboarding",
      "attributes": {
        "unique_id": "onboarding-uuid-456",
        "name": "New User Onboarding",
        "description": "Welcome flow for new users",
        "status": "active",
        "steps_count": 5,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 12
  }
}
```

---

### GET /onboardings/:unique_id/ - Get Onboarding with Steps

Retrieves a single onboarding definition including all its steps.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/onboardings/onboarding-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "onboarding-uuid-456",
    "type": "onboarding",
    "attributes": {
      "unique_id": "onboarding-uuid-456",
      "name": "New User Onboarding",
      "description": "Welcome flow for new users",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z",
      "steps": [
        {
          "unique_id": "step-uuid-001",
          "name": "Profile Setup",
          "description": "Complete your user profile",
          "order": 1,
          "step_type": "action",
          "required": true,
          "metadata": {"target_url": "/profile/setup"}
        },
        {
          "unique_id": "step-uuid-002",
          "name": "Verify Email",
          "description": "Confirm your email address",
          "order": 2,
          "step_type": "verification",
          "required": true,
          "metadata": {}
        },
        {
          "unique_id": "step-uuid-003",
          "name": "Welcome Tour",
          "description": "Take the platform tour",
          "order": 3,
          "step_type": "information",
          "required": false,
          "metadata": {"tour_id": "welcome-tour"}
        }
      ]
    }
  }
}
```

**Errors:**
- `404 Not Found` - Onboarding not found

---

### POST /onboardings/ - Create Onboarding

Creates a new onboarding definition. **Requires role_id 2** (admin role).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/onboardings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "onboarding": {
      "name": "New User Onboarding",
      "description": "Welcome flow for new users"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Onboarding name |
| `description` | string | No | Onboarding description |

**Response 201:**
```json
{
  "data": {
    "id": "onboarding-uuid-456",
    "type": "onboarding",
    "attributes": {
      "unique_id": "onboarding-uuid-456",
      "name": "New User Onboarding",
      "description": "Welcome flow for new users",
      "status": "active",
      "steps": [],
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Insufficient permissions (role_id must be 2)
- `422 Unprocessable Entity` - Validation errors

---

### PUT /onboardings/:unique_id/steps - Add Step

Adds a new step to an onboarding definition.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/onboardings/onboarding-uuid-456/steps" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "step": {
      "name": "Profile Setup",
      "description": "Complete your user profile",
      "order": 1,
      "step_type": "action",
      "required": true,
      "metadata": {"target_url": "/profile/setup"}
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Step name |
| `description` | string | No | Step description |
| `order` | integer | Yes | Step order (position in sequence) |
| `step_type` | string | Yes | Step type (action, verification, information) |
| `required` | boolean | No | Whether step is required (default: true) |
| `metadata` | object | No | Custom step metadata |

**Response 200:**
```json
{
  "data": {
    "id": "onboarding-uuid-456",
    "type": "onboarding",
    "attributes": {
      "unique_id": "onboarding-uuid-456",
      "name": "New User Onboarding",
      "steps": [
        {
          "unique_id": "step-uuid-001",
          "name": "Profile Setup",
          "description": "Complete your user profile",
          "order": 1,
          "step_type": "action",
          "required": true,
          "metadata": {"target_url": "/profile/setup"}
        }
      ],
      "updated_at": "2025-01-12T11:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Onboarding not found
- `422 Unprocessable Entity` - Validation errors

---

### PUT /onboardings/:unique_id/steps/:step_unique_id - Update Step

Updates an existing step in an onboarding definition.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/onboardings/onboarding-uuid-456/steps/step-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "step": {
      "name": "Complete Profile",
      "description": "Fill in your profile details",
      "required": true,
      "metadata": {"target_url": "/profile/complete", "timeout_minutes": 30}
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | No | Updated step name |
| `description` | string | No | Updated description |
| `order` | integer | No | Updated order |
| `step_type` | string | No | Updated step type |
| `required` | boolean | No | Updated required flag |
| `metadata` | object | No | Updated metadata |

**Response 200:**
```json
{
  "data": {
    "id": "onboarding-uuid-456",
    "type": "onboarding",
    "attributes": {
      "unique_id": "onboarding-uuid-456",
      "name": "New User Onboarding",
      "steps": [
        {
          "unique_id": "step-uuid-001",
          "name": "Complete Profile",
          "description": "Fill in your profile details",
          "order": 1,
          "step_type": "action",
          "required": true,
          "metadata": {"target_url": "/profile/complete", "timeout_minutes": 30}
        }
      ],
      "updated_at": "2025-01-12T12:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Onboarding or step not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /onboardings/:unique_id/steps/:step_unique_id - Delete Step

Deletes a step from an onboarding definition.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/onboardings/onboarding-uuid-456/steps/step-uuid-003" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Onboarding or step not found

---

### PUT /onboardings/:unique_id/users/:user_unique_id/ - Admin Progress Step for User

Admin endpoint to progress a user to the next step in an onboarding. Used for manual intervention or automated progression.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/onboardings/onboarding-uuid-456/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "journey": {
      "metadata": {"admin_note": "Manually progressed by support"}
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metadata` | object | No | Additional metadata for the progression |

**Response 200:**
```json
{
  "data": {
    "id": "journey-uuid-789",
    "type": "journey",
    "attributes": {
      "unique_id": "journey-uuid-789",
      "onboarding_id": "onboarding-uuid-456",
      "user_unique_id": "user-uuid-123",
      "current_step": 3,
      "status": "active",
      "started_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Onboarding or user not found
- `422 Unprocessable Entity` - Journey already completed

---

## Data Models

### Onboarding
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Onboarding name |
| `description` | string | Onboarding description |
| `steps` | array | Ordered array of Step objects |
| `status` | enum | active, inactive |
| `steps_count` | integer | Number of steps |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Step
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Step name |
| `description` | string | Step description |
| `order` | integer | Position in sequence |
| `step_type` | string | Type: action, verification, information |
| `required` | boolean | Whether step is mandatory |
| `metadata` | object | Custom step configuration |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "403",
    "code": "forbidden",
    "title": "Insufficient Permissions",
    "detail": "Creating onboardings requires role_id 2."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-onboarding
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
// OnboardingsService — client.onboarding.onboardings
client.onboarding.onboardings.list(params?: ListOnboardingsParams): Promise<PageResult<Onboarding>>;
client.onboarding.onboardings.get(uniqueId: string): Promise<Onboarding>;
client.onboarding.onboardings.create(data: CreateOnboardingRequest): Promise<Onboarding>;
client.onboarding.onboardings.update(uniqueId: string, data: UpdateOnboardingRequest): Promise<Onboarding>;
client.onboarding.onboardings.delete(uniqueId: string): Promise<void>;
client.onboarding.onboardings.addStep(uniqueId: string, data: AddStepRequest): Promise<OnboardingStep>;
client.onboarding.onboardings.updateStep(uniqueId: string, stepUniqueId: string, data: UpdateStepRequest): Promise<OnboardingStep>;
client.onboarding.onboardings.deleteStep(uniqueId: string, stepUniqueId: string): Promise<void>;
client.onboarding.onboardings.stepUser(uniqueId: string, userUniqueId: string, stepData?: Record<string, unknown>): Promise<Onboarding>;
```

### TypeScript Types

```typescript
import type {
  Onboarding,
  CreateOnboardingRequest,
  UpdateOnboardingRequest,
  ListOnboardingsParams,
  OnboardingStep,
  AddStepRequest,
  UpdateStepRequest,
} from '@23blocks/block-onboarding';
```

### React Hook

```typescript
import { useOnboardingBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useOnboardingBlock();
  const result = await client.onboarding.onboardings.list({ page: 1, perPage: 20 });
}
```
