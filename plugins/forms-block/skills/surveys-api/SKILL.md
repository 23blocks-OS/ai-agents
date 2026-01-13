---
name: surveys-api
description: Manage 23blocks survey instances via REST API. Use for feedback collection, NPS surveys, and data gathering with magic link access.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Surveys API

Complete API reference for 23blocks survey instance management.

## Base URL
```
https://forms.23blocks.com
```

## Authentication
```bash
Authorization: Bearer {access_token}
AppId: {api_access_key}
Content-Type: application/json
```

---

## Endpoints

### GET /surveys/:form_unique_id/instances - List Survey Instances

Lists survey instances for a specific survey form.

**Request:**
```bash
curl -X GET "$API_URL/surveys/form-123/instances?page=1&records=20" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
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
curl -X GET "$API_URL/surveys/form-123/instances/status/completed" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
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
curl -X GET "$API_URL/surveys/form-123/instances/survey-inst-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
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
curl -X POST "$API_URL/surveys/form-123/instances" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
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
curl -X PUT "$API_URL/surveys/form-123/instances/survey-inst-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
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
curl -X PUT "$API_URL/surveys/form-123/instances/survey-inst-123/status" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
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
curl -X POST "$API_URL/surveys/form-123/instances/survey-inst-123/resend_magic_link" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
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
curl -X DELETE "$API_URL/surveys/form-123/instances/survey-inst-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 204:** No content

---

### POST /surveys/users - List by User

Lists all surveys for a specific user.

**Request:**
```bash
curl -X POST "$API_URL/surveys/users" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
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
