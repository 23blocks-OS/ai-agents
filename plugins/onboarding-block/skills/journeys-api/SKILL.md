---
name: onboarding-journeys-api
description: Manage 23blocks user onboarding journeys via REST API. Use when starting journeys, progressing through steps, logging step completions, viewing journey status and details, or suspending and resuming journeys.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Journeys API

Complete API reference for 23blocks user-facing journey progress through onboarding steps.

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
curl -X GET "$BLOCKS_API_URL/onboard/journey-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /onboard/:unique_id/start - Start Journey

Starts a new onboarding journey for the authenticated user. The `:unique_id` refers to the onboarding definition ID.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/onboard/onboarding-uuid-456/start" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "journey": {
      "metadata": {"source": "welcome_page"}
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metadata` | object | No | Initial journey metadata |

**Response 201:**
```json
{
  "data": {
    "id": "journey-uuid-789",
    "type": "journey",
    "attributes": {
      "unique_id": "journey-uuid-789",
      "onboarding_id": "onboarding-uuid-456",
      "user_unique_id": "user-uuid-123",
      "current_step": 1,
      "status": "active",
      "started_at": "2025-01-12T10:30:00Z",
      "completed_at": null,
      "logs": []
    }
  }
}
```

**Errors:**
- `404 Not Found` - Onboarding not found
- `422 Unprocessable Entity` - User already has active journey for this onboarding

---

### GET /onboard/:unique_id - Get Journey Status

Retrieves the current status of a journey. The `:unique_id` refers to the journey ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/onboard/journey-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

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
      "current_step": 2,
      "status": "active",
      "started_at": "2025-01-10T10:30:00Z",
      "completed_at": null
    }
  }
}
```

**Errors:**
- `404 Not Found` - Journey not found

---

### GET /onboard/:unique_id/details - Get Detailed Journey with Logs

Retrieves a journey with full step details and completion logs.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/onboard/journey-uuid-789/details" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

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
      "current_step": 2,
      "status": "active",
      "started_at": "2025-01-10T10:30:00Z",
      "completed_at": null,
      "logs": [
        {
          "step_unique_id": "step-uuid-001",
          "step_name": "Profile Setup",
          "step_order": 1,
          "action": "completed",
          "metadata": {"completed_action": "profile_saved"},
          "logged_at": "2025-01-10T11:00:00Z"
        },
        {
          "step_unique_id": "step-uuid-002",
          "step_name": "Verify Email",
          "step_order": 2,
          "action": "started",
          "metadata": {},
          "logged_at": "2025-01-10T11:00:01Z"
        }
      ]
    },
    "relationships": {
      "onboarding": {
        "data": {
          "id": "onboarding-uuid-456",
          "type": "onboarding"
        }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Journey not found

---

### PUT /onboard/:unique_id - Progress to Next Step

Marks the current step as completed and advances the journey to the next step. If the current step is the last step, the journey status changes to `completed`.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/onboard/journey-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "journey": {
      "metadata": {"completed_action": "email_verified"}
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metadata` | object | No | Metadata about the step completion |

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
      "completed_at": null
    }
  }
}
```

**Response 200 (journey completed):**
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
      "status": "completed",
      "started_at": "2025-01-10T10:30:00Z",
      "completed_at": "2025-01-12T15:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Journey not found
- `422 Unprocessable Entity` - Journey already completed or suspended

---

### PUT /onboard/:unique_id/log - Log Step Without Progressing

Logs a step action without advancing the journey to the next step. Useful for tracking intermediate actions within a step.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/onboard/journey-uuid-789/log" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "log": {
      "action": "field_completed",
      "metadata": {"field": "company_name", "value": "Acme Corp"}
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | Action being logged |
| `metadata` | object | No | Additional metadata for the log entry |

**Response 200:**
```json
{
  "data": {
    "id": "journey-uuid-789",
    "type": "journey",
    "attributes": {
      "unique_id": "journey-uuid-789",
      "current_step": 2,
      "status": "active",
      "logs": [
        {
          "step_unique_id": "step-uuid-002",
          "step_name": "Verify Email",
          "step_order": 2,
          "action": "field_completed",
          "metadata": {"field": "company_name", "value": "Acme Corp"},
          "logged_at": "2025-01-12T11:30:00Z"
        }
      ]
    }
  }
}
```

**Errors:**
- `404 Not Found` - Journey not found
- `422 Unprocessable Entity` - Journey not active

---

### PUT /onboard/:unique_id/suspend - Suspend Journey

Suspends an active journey. Suspended journeys can be resumed later.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/onboard/journey-uuid-789/suspend" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "journey": {
      "metadata": {"reason": "user_requested_pause"}
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metadata` | object | No | Reason or context for suspension |

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
      "current_step": 2,
      "status": "suspended",
      "started_at": "2025-01-10T10:30:00Z",
      "completed_at": null
    }
  }
}
```

**Errors:**
- `404 Not Found` - Journey not found
- `422 Unprocessable Entity` - Journey not active

---

### PUT /onboard/:unique_id/resume - Resume Journey

Resumes a previously suspended journey from where it was paused.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/onboard/journey-uuid-789/resume" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "journey": {
      "metadata": {"resumed_from": "remarketing_email"}
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metadata` | object | No | Context for resumption |

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
      "current_step": 2,
      "status": "active",
      "started_at": "2025-01-10T10:30:00Z",
      "completed_at": null
    }
  }
}
```

**Errors:**
- `404 Not Found` - Journey not found
- `422 Unprocessable Entity` - Journey not suspended

---

## Data Models

### Journey
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `onboarding_id` | uuid | Associated onboarding definition |
| `user_unique_id` | uuid | Associated user |
| `current_step` | integer | Current step number (order) |
| `status` | enum | active, completed, suspended |
| `started_at` | timestamp | Journey start time |
| `completed_at` | timestamp | Journey completion time (null if not completed) |
| `logs` | array | Array of JourneyLog entries |

### JourneyLog
| Field | Type | Description |
|-------|------|-------------|
| `step_unique_id` | uuid | Associated step |
| `step_name` | string | Step name at time of log |
| `step_order` | integer | Step order at time of log |
| `action` | string | Action performed (started, completed, field_completed, etc.) |
| `metadata` | object | Additional log metadata |
| `logged_at` | timestamp | Log entry time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "unprocessable_entity",
    "title": "Journey Not Active",
    "detail": "Cannot progress a journey that is not in active status."
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
// OnboardService — client.onboarding.onboard
client.onboarding.onboard.start(uniqueId: string, data?: StartOnboardRequest): Promise<OnboardJourney>;
client.onboarding.onboard.get(uniqueId: string): Promise<OnboardJourney>;
client.onboarding.onboard.getDetails(uniqueId: string): Promise<OnboardJourneyDetails>;
client.onboarding.onboard.step(uniqueId: string, data: StepOnboardRequest): Promise<OnboardJourney>;
client.onboarding.onboard.log(uniqueId: string, data: LogOnboardRequest): Promise<OnboardJourney>;
client.onboarding.onboard.suspend(uniqueId: string): Promise<OnboardJourney>;
client.onboarding.onboard.resume(uniqueId: string): Promise<OnboardJourney>;

// UserJourneysService — client.onboarding.userJourneys
client.onboarding.userJourneys.list(params?: ListUserJourneysParams): Promise<PageResult<UserJourney>>;
client.onboarding.userJourneys.get(uniqueId: string): Promise<UserJourney>;
client.onboarding.userJourneys.start(data: StartJourneyRequest): Promise<UserJourney>;
client.onboarding.userJourneys.completeStep(uniqueId: string, data: CompleteStepRequest): Promise<UserJourney>;
client.onboarding.userJourneys.abandon(uniqueId: string): Promise<UserJourney>;
client.onboarding.userJourneys.getByUser(userUniqueId: string): Promise<UserJourney[]>;
client.onboarding.userJourneys.getProgress(uniqueId: string): Promise<{ progress: number; currentStep?: number; completedSteps?: number[] }>;
client.onboarding.userJourneys.listByUserAndOnboarding(userUniqueId: string, onboardingUniqueId: string): Promise<UserJourney>;
client.onboarding.userJourneys.reportList(params: UserJourneyReportParams): Promise<UserJourneyReportList>;
client.onboarding.userJourneys.reportSummary(params: UserJourneyReportParams): Promise<UserJourneyReportSummary>;
```

### TypeScript Types

```typescript
import type {
  OnboardJourney,
  OnboardJourneyDetails,
  OnboardStepDetail,
  OnboardLogEntry,
  StartOnboardRequest,
  StepOnboardRequest,
  LogOnboardRequest,
  UserJourney,
  StartJourneyRequest,
  CompleteStepRequest,
  ListUserJourneysParams,
  UserJourneyReportParams,
  UserJourneyReportSummary,
  UserJourneyReportList,
} from '@23blocks/block-onboarding';
```

### React Hook

```typescript
import { useOnboardingBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useOnboardingBlock();
  const journey = await client.onboarding.onboard.start('onboarding-uuid');
}
```
