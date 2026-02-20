---
name: onboarding-remarketing-api
description: Manage 23blocks onboarding remarketing via REST API. Use when triggering notifications for users with abandoned onboarding journeys based on configurable elapsed time thresholds.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Remarketing API

Complete API reference for 23blocks onboarding remarketing tools to re-engage users with abandoned journeys.

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
curl -X GET "$BLOCKS_API_URL/tools/remarketing/abandoned_journeys" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /tools/remarketing/abandoned_journeys - Notify Abandoned Journeys

Triggers notifications for users who have abandoned their onboarding journeys. A journey is considered abandoned when it has been active but idle for more than the specified elapsed hours. This endpoint identifies abandoned journeys and triggers the configured notification mechanism (e.g., email via Mandrill templates).

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tools/remarketing/abandoned_journeys?elapsed_hours=24" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `elapsed_hours` | integer | No | Hours of inactivity threshold (default: 24) |

**Response 200:**
```json
{
  "data": {
    "id": "remarketing-run-uuid",
    "type": "remarketing_result",
    "attributes": {
      "elapsed_hours": 24,
      "abandoned_journeys_found": 15,
      "notifications_sent": 12,
      "notifications_skipped": 3,
      "skipped_reasons": [
        {
          "journey_id": "journey-uuid-101",
          "reason": "user_email_not_verified"
        },
        {
          "journey_id": "journey-uuid-102",
          "reason": "notification_already_sent_recently"
        },
        {
          "journey_id": "journey-uuid-103",
          "reason": "user_opted_out"
        }
      ],
      "journeys": [
        {
          "journey_id": "journey-uuid-201",
          "user_unique_id": "user-uuid-301",
          "user_email": "user1@example.com",
          "onboarding_name": "New User Onboarding",
          "current_step": 2,
          "total_steps": 5,
          "last_activity_at": "2025-01-10T10:30:00Z",
          "notification_status": "sent"
        },
        {
          "journey_id": "journey-uuid-202",
          "user_unique_id": "user-uuid-302",
          "user_email": "user2@example.com",
          "onboarding_name": "New User Onboarding",
          "current_step": 1,
          "total_steps": 5,
          "last_activity_at": "2025-01-09T15:00:00Z",
          "notification_status": "sent"
        }
      ],
      "executed_at": "2025-01-12T15:00:00Z"
    }
  }
}
```

**Example - 48 hour threshold:**
```bash
curl -X GET "$BLOCKS_API_URL/tools/remarketing/abandoned_journeys?elapsed_hours=48" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Example - 72 hour threshold:**
```bash
curl -X GET "$BLOCKS_API_URL/tools/remarketing/abandoned_journeys?elapsed_hours=72" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Errors:**
- `422 Unprocessable Entity` - Invalid elapsed_hours value

---

## Data Models

### RemarketingResult
| Field | Type | Description |
|-------|------|-------------|
| `elapsed_hours` | integer | Time threshold used for the query |
| `abandoned_journeys_found` | integer | Total abandoned journeys found |
| `notifications_sent` | integer | Notifications successfully sent |
| `notifications_skipped` | integer | Notifications skipped |
| `skipped_reasons` | array | Reasons for skipped notifications |
| `journeys` | array | List of abandoned journey details |
| `executed_at` | timestamp | When the remarketing run was executed |

### AbandonedJourney
| Field | Type | Description |
|-------|------|-------------|
| `journey_id` | uuid | Journey identifier |
| `user_unique_id` | uuid | User identifier |
| `user_email` | string | User email address |
| `onboarding_name` | string | Onboarding name |
| `current_step` | integer | Step where user stopped |
| `total_steps` | integer | Total steps in onboarding |
| `last_activity_at` | timestamp | Last activity timestamp |
| `notification_status` | string | sent, skipped |

### SkippedReason
| Field | Type | Description |
|-------|------|-------------|
| `journey_id` | uuid | Journey identifier |
| `reason` | string | Reason for skipping (user_email_not_verified, notification_already_sent_recently, user_opted_out) |

---

## Best Practices

- **Start with 24 hours**: Use the default threshold for first-pass remarketing
- **Escalate gradually**: Run at 24h, 48h, and 72h intervals for multi-touch campaigns
- **Pair with mail templates**: Configure Mandrill templates for personalized remarketing emails
- **Monitor skip reasons**: High skip rates may indicate data quality issues
- **Schedule regularly**: Set up recurring calls via cron or workflow automation

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "unprocessable_entity",
    "title": "Invalid Parameter",
    "detail": "elapsed_hours must be a positive integer."
  }]
}
```
