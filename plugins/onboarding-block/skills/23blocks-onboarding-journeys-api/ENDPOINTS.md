# Journeys API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

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
