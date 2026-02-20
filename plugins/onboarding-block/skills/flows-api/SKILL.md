---
name: onboarding-flows-api
description: Manage 23blocks onboarding flows via REST API. Use when retrieving flows by source or progressing steps within a source-linked flow.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Flows API

Complete API reference for 23blocks onboarding source-linked flow management and step progression.

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
curl -X GET "$BLOCKS_API_URL/flows/flow-uuid-123/sources/source-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /flows/:unique_id/sources/:source_unique_id - Get Flows by Source

Retrieves flows associated with a specific source within a flow definition.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/flows/flow-uuid-123/sources/source-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY" \
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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Flow Not Found",
    "detail": "The requested flow or source could not be found."
  }]
}
```
