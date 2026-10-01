---
name: 23blocks-onboarding-flows-api
description: "Onboarding Block source-linked flows: get by source, progress steps. Use when a flow is tied to an external source."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Flows API

Complete API reference for 23blocks onboarding source-linked flow management and step progression.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://onboarding.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /flows/:unique_id/sources/:source_unique_id - Get Flows by Source

Retrieves flows associated with a specific source within a flow definition.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/flows/flow-uuid-123/sources/source-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "flow-uuid-123",
    "type": "flow",
    "attributes": {
      "unique_id": "flow-uuid-123",
      "source_unique_id": "source-uuid-456",
      "onboarding_id": "onboarding-uuid-789",
      "current_step": 2,
      "total_steps": 5,
      "status": "active",
      "metadata": {
        "source_type": "landing_page",
        "campaign": "summer_promo"
      },
      "steps": [
        {
          "unique_id": "step-uuid-001",
          "name": "Welcome",
          "order": 1,
          "status": "completed"
        },
        {
          "unique_id": "step-uuid-002",
          "name": "Profile Setup",
          "order": 2,
          "status": "active"
        },
        {
          "unique_id": "step-uuid-003",
          "name": "Preferences",
          "order": 3,
          "status": "pending"
        }
      ],
      "started_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T11:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Flow or source not found

---

### PUT /flows/:unique_id/sources/:source_unique_id - Progress Step in Flow

Advances the flow to the next step for the specified source.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/flows/flow-uuid-123/sources/source-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "flow": {
      "metadata": {"completed_action": "profile_saved", "source_data": {"ref": "landing-page-v2"}}
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metadata` | object | No | Metadata about the step progression |

**Response 200:**
```json
{
  "data": {
    "id": "flow-uuid-123",
    "type": "flow",
    "attributes": {
      "unique_id": "flow-uuid-123",
      "source_unique_id": "source-uuid-456",
      "onboarding_id": "onboarding-uuid-789",
      "current_step": 3,
      "total_steps": 5,
      "status": "active",
      "metadata": {
        "source_type": "landing_page",
        "campaign": "summer_promo"
      },
      "started_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Response 200 (flow completed):**
```json
{
  "data": {
    "id": "flow-uuid-123",
    "type": "flow",
    "attributes": {
      "unique_id": "flow-uuid-123",
      "source_unique_id": "source-uuid-456",
      "onboarding_id": "onboarding-uuid-789",
      "current_step": 5,
      "total_steps": 5,
      "status": "completed",
      "started_at": "2025-01-10T10:30:00Z",
      "completed_at": "2025-01-12T14:30:00Z",
      "updated_at": "2025-01-12T14:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Flow or source not found
- `422 Unprocessable Entity` - Flow already completed

---

## Data Models

### Flow
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Flow unique identifier |
| `source_unique_id` | uuid | Associated source identifier |
| `onboarding_id` | uuid | Associated onboarding definition |
| `current_step` | integer | Current step number |
| `total_steps` | integer | Total number of steps |
| `status` | enum | active, completed |
| `metadata` | object | Flow metadata (source_type, campaign, etc.) |
| `steps` | array | Array of flow step statuses |
| `started_at` | timestamp | Flow start time |
| `completed_at` | timestamp | Flow completion time |
| `updated_at` | timestamp | Last update time |

### FlowStep
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Step unique identifier |
| `name` | string | Step name |
| `order` | integer | Step order |
| `status` | enum | pending, active, completed |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Flow Not Found","detail":"The requested flow or source could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-onboarding`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// FlowsService — client.onboarding.flows
client.onboarding.flows.list(params?: ListFlowsParams): Promise<PageResult<Flow>>;
client.onboarding.flows.get(uniqueId: string): Promise<Flow>;
client.onboarding.flows.create(data: CreateFlowRequest): Promise<Flow>;
client.onboarding.flows.update(uniqueId: string, data: UpdateFlowRequest): Promise<Flow>;
client.onboarding.flows.delete(uniqueId: string): Promise<void>;
client.onboarding.flows.listByOnboarding(onboardingUniqueId: string): Promise<Flow[]>;
client.onboarding.flows.getBySource(uniqueId: string, sourceUniqueId: string): Promise<Flow>;
client.onboarding.flows.stepBySource(uniqueId: string, sourceUniqueId: string, stepData?: Record<string, unknown>): Promise<Flow>;
```

### TypeScript Types

```typescript
import type {
  Flow,
  CreateFlowRequest,
  UpdateFlowRequest,
  ListFlowsParams,
} from '@23blocks/block-onboarding';
```

### React Hook

```typescript
import { useOnboardingBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useOnboardingBlock();
  const result = await client.onboarding.flows.list({ page: 1, perPage: 20 });
}
```
