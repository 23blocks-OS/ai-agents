---
name: onboarding-reports-api
description: Manage 23blocks onboarding journey reports via REST API. Use when viewing user journey history, retrieving specific journey details, listing journeys with filters, or generating summary reports on onboarding progress.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Reports API

Complete API reference for 23blocks onboarding journey reporting with filtering and summary analytics.

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
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/onboardings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /users/:unique_id/onboardings - Get User's Journeys

Retrieves all onboarding journeys for a specific user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/onboardings?page=1&records=20" \
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
      "id": "journey-uuid-789",
      "type": "journey",
      "attributes": {
        "unique_id": "journey-uuid-789",
        "onboarding_id": "onboarding-uuid-456",
        "onboarding_name": "New User Onboarding",
        "user_unique_id": "user-uuid-123",
        "current_step": 3,
        "total_steps": 5,
        "status": "active",
        "started_at": "2025-01-10T10:30:00Z",
        "completed_at": null
      }
    },
    {
      "id": "journey-uuid-790",
      "type": "journey",
      "attributes": {
        "unique_id": "journey-uuid-790",
        "onboarding_id": "onboarding-uuid-457",
        "onboarding_name": "Premium Feature Tour",
        "user_unique_id": "user-uuid-123",
        "current_step": 4,
        "total_steps": 4,
        "status": "completed",
        "started_at": "2025-01-08T09:00:00Z",
        "completed_at": "2025-01-09T12:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 2
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### GET /users/:unique_id/onboardings/:onboarding_unique_id - Get Specific Journey

Retrieves a specific onboarding journey for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/onboardings/onboarding-uuid-456" \
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
      "onboarding_name": "New User Onboarding",
      "user_unique_id": "user-uuid-123",
      "current_step": 3,
      "total_steps": 5,
      "status": "active",
      "started_at": "2025-01-10T10:30:00Z",
      "completed_at": null,
      "logs": [
        {
          "step_unique_id": "step-uuid-001",
          "step_name": "Profile Setup",
          "step_order": 1,
          "action": "completed",
          "logged_at": "2025-01-10T11:00:00Z"
        },
        {
          "step_unique_id": "step-uuid-002",
          "step_name": "Verify Email",
          "step_order": 2,
          "action": "completed",
          "logged_at": "2025-01-10T14:00:00Z"
        }
      ]
    }
  }
}
```

**Errors:**
- `404 Not Found` - User or onboarding journey not found

---

### POST /reports/user_journeys/list - List Journeys with Filters

Lists journeys with advanced filtering. Requires scope `user_journeys:reports:list`.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/user_journeys/list" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "filters": {
        "onboarding_id": "onboarding-uuid-456",
        "status": "active",
        "started_after": "2025-01-01T00:00:00Z",
        "started_before": "2025-01-31T23:59:59Z"
      },
      "page": 1,
      "records": 20,
      "sort_by": "started_at",
      "sort_order": "desc"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filters.onboarding_id` | uuid | No | Filter by onboarding ID |
| `filters.status` | string | No | Filter by status (active, completed, suspended) |
| `filters.started_after` | timestamp | No | Filter journeys started after date |
| `filters.started_before` | timestamp | No | Filter journeys started before date |
| `filters.user_unique_id` | uuid | No | Filter by user ID |
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `sort_by` | string | No | Sort field (started_at, completed_at, current_step) |
| `sort_order` | string | No | Sort direction (asc, desc) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "journey-uuid-789",
      "type": "journey_report",
      "attributes": {
        "unique_id": "journey-uuid-789",
        "onboarding_id": "onboarding-uuid-456",
        "onboarding_name": "New User Onboarding",
        "user_unique_id": "user-uuid-123",
        "user_email": "user@example.com",
        "user_display_name": "John Doe",
        "current_step": 3,
        "total_steps": 5,
        "completion_percentage": 60,
        "status": "active",
        "started_at": "2025-01-10T10:30:00Z",
        "completed_at": null
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 87,
    "scope": "user_journeys:reports:list"
  }
}
```

**Errors:**
- `403 Forbidden` - Missing required scope `user_journeys:reports:list`
- `422 Unprocessable Entity` - Invalid filter parameters

---

### POST /reports/user_journeys/summary - Summary Reports

Generates aggregated summary statistics for onboarding journeys. Requires scope `user_journeys:reports:summary`.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/user_journeys/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "filters": {
        "onboarding_id": "onboarding-uuid-456",
        "started_after": "2025-01-01T00:00:00Z",
        "started_before": "2025-01-31T23:59:59Z"
      },
      "group_by": "status"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filters.onboarding_id` | uuid | No | Filter by onboarding ID |
| `filters.started_after` | timestamp | No | Filter journeys started after date |
| `filters.started_before` | timestamp | No | Filter journeys started before date |
| `group_by` | string | No | Group results by field (status, onboarding_id, current_step) |

**Response 200:**
```json
{
  "data": {
    "id": "summary-report",
    "type": "journey_summary",
    "attributes": {
      "total_journeys": 150,
      "active_journeys": 45,
      "completed_journeys": 87,
      "suspended_journeys": 18,
      "completion_rate": 58.0,
      "average_completion_time_hours": 72.5,
      "average_steps_completed": 3.2,
      "breakdown": [
        {
          "status": "active",
          "count": 45,
          "percentage": 30.0
        },
        {
          "status": "completed",
          "count": 87,
          "percentage": 58.0
        },
        {
          "status": "suspended",
          "count": 18,
          "percentage": 12.0
        }
      ]
    },
    "meta": {
      "scope": "user_journeys:reports:summary",
      "generated_at": "2025-01-12T15:00:00Z"
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Missing required scope `user_journeys:reports:summary`
- `422 Unprocessable Entity` - Invalid filter parameters

---

## Data Models

### JourneyReport
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Journey unique identifier |
| `onboarding_id` | uuid | Associated onboarding |
| `onboarding_name` | string | Onboarding name |
| `user_unique_id` | uuid | Associated user |
| `user_email` | string | User email |
| `user_display_name` | string | User display name |
| `current_step` | integer | Current step number |
| `total_steps` | integer | Total steps in onboarding |
| `completion_percentage` | float | Percentage of steps completed |
| `status` | enum | active, completed, suspended |
| `started_at` | timestamp | Journey start time |
| `completed_at` | timestamp | Journey completion time |

### JourneySummary
| Field | Type | Description |
|-------|------|-------------|
| `total_journeys` | integer | Total journey count |
| `active_journeys` | integer | Active journey count |
| `completed_journeys` | integer | Completed journey count |
| `suspended_journeys` | integer | Suspended journey count |
| `completion_rate` | float | Completion rate percentage |
| `average_completion_time_hours` | float | Average time to complete |
| `average_steps_completed` | float | Average steps completed |
| `breakdown` | array | Status breakdown with counts and percentages |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "403",
    "code": "forbidden",
    "title": "Insufficient Scope",
    "detail": "This endpoint requires the user_journeys:reports:list scope."
  }]
}
```
