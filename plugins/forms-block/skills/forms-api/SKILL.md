---
name: forms-api
description: Create and manage 23blocks form definitions via REST API. Use when creating forms, configuring form types, or managing form settings.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Forms API

Complete API reference for 23blocks form management.

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

### GET /forms - List Forms

Lists all forms with pagination and filtering.

**Request:**
```bash
curl -X GET "$API_URL/forms?page=1&records=20&search=contact" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `search` | string | No | Search by name or description |
| `sort` | string | No | Sort field (prefix with `-` for desc) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "9ebed093-574a-4be4-9076-cfca36493330",
      "type": "form",
      "attributes": {
        "unique_id": "9ebed093-574a-4be4-9076-cfca36493330",
        "name": "Contact Form",
        "form_type": "landing",
        "description": "Main contact form",
        "form_url": "https://example.com/contact",
        "status": "active",
        "require_otp_verification": false,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      },
      "relationships": {
        "form_schema": {
          "data": null
        }
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  },
  "links": {
    "self": "/forms/?search=&order=ASC&page=1&records=20",
    "next": "/forms/?search=&order=ASC&page=2&records=20",
    "prev": "/forms/?search=&order=ASC&page=0&records=20"
  }
}
```

---

### GET /forms/:unique_id - Get Form

Retrieves a form by unique ID with its schema.

**Request:**
```bash
curl -X GET "$API_URL/forms/9ebed093-574a-4be4-9076-cfca36493330" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 200:**
```json
{
  "data": {
    "id": "9ebed093-574a-4be4-9076-cfca36493330",
    "type": "form",
    "attributes": {
      "unique_id": "9ebed093-574a-4be4-9076-cfca36493330",
      "name": "GAD-7 Assessment",
      "form_type": "app_form",
      "description": "Generalized Anxiety Disorder assessment",
      "form_url": null,
      "form_domain": null,
      "form_fields": null,
      "status": "active",
      "require_otp_verification": true,
      "send_confirmation_mail": true,
      "send_admin_notification": false,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "form_schema": {
        "data": {
          "id": "abc123",
          "type": "form_schema"
        }
      }
    }
  },
  "included": [
    {
      "id": "abc123",
      "type": "form_schema",
      "attributes": {
        "name": "GAD-7",
        "description": "Generalized Anxiety Disorder 7-item scale"
      },
      "relationships": {
        "form_schema_versions": {
          "data": [
            { "id": "v1", "type": "form_schema_version" }
          ]
        }
      }
    }
  ]
}
```

**Errors:**
- `404 Not Found` - Form not found

---

### POST /forms - Create Form

Creates a new form definition.

**Request:**
```bash
curl -X POST "$API_URL/forms" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "form": {
      "name": "Patient Intake Form",
      "form_type": "app_form",
      "description": "New patient information collection",
      "require_otp_verification": true,
      "send_confirmation_mail": true,
      "form_schema_unique_id": "existing-schema-id"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Form name |
| `form_type` | enum | Yes | landing, survey, appointment, subscription, referral, app_form |
| `description` | string | No | Form description |
| `form_url` | string | No | External form URL |
| `form_domain` | string | No | Domain for form (validated against company domain) |
| `form_fields` | json | No | Legacy field definitions |
| `require_otp_verification` | boolean | No | Require email OTP verification (default: false) |
| `send_confirmation_mail` | boolean | No | Send confirmation email on submission |
| `mail_template` | json | No | Email template configuration |
| `send_admin_notification` | boolean | No | Notify admin on submission |
| `admin_notification_email` | string | No | Admin email for notifications |
| `form_schema_unique_id` | string | No | Link to existing form schema (for app_form type) |
| `success_url` | string | No | Redirect URL on success |
| `error_url` | string | No | Redirect URL on error |

**Response 201:**
```json
{
  "data": {
    "id": "new-form-id",
    "type": "form",
    "attributes": {
      "unique_id": "new-form-id",
      "name": "Patient Intake Form",
      "form_type": "app_form",
      "status": "active",
      "require_otp_verification": true,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `400 frm_10110` - Invalid form domain (must match company domain)
- `422 Unprocessable Entity` - Validation errors

---

### PUT /forms/:unique_id - Update Form

Updates an existing form.

**Request:**
```bash
curl -X PUT "$API_URL/forms/9ebed093-574a-4be4-9076-cfca36493330" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "form": {
      "name": "Updated Form Name",
      "description": "Updated description",
      "require_otp_verification": true
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "9ebed093-574a-4be4-9076-cfca36493330",
    "type": "form",
    "attributes": {
      "unique_id": "9ebed093-574a-4be4-9076-cfca36493330",
      "name": "Updated Form Name",
      "description": "Updated description",
      "require_otp_verification": true,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Form not found
- `400 frm_10110` - Invalid form domain

---

### DELETE /forms/:unique_id - Delete Form

Soft-deletes a form (sets status to deleted, enabled to false).

**Request:**
```bash
curl -X DELETE "$API_URL/forms/9ebed093-574a-4be4-9076-cfca36493330" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Form not found

---

### GET /forms/types/:unique_id - List Forms by Type

Lists forms filtered by form type ID.

**Request:**
```bash
curl -X GET "$API_URL/forms/types/app_form?page=1&records=20" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 200:** Same as GET /forms

---

## Data Models

### Form
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Form name |
| `form_type` | enum | landing, survey, appointment, subscription, referral, app_form |
| `description` | string | Form description |
| `form_url` | string | External form URL |
| `form_domain` | string | Domain restriction |
| `form_fields` | json | Legacy field definitions |
| `status` | enum | active, deleted |
| `require_otp_verification` | boolean | Require email OTP |
| `send_confirmation_mail` | boolean | Send confirmation email |
| `mail_template` | json | Email template config |
| `send_admin_notification` | boolean | Notify admin |
| `admin_notification_email` | string | Admin email |
| `success_url` | string | Success redirect |
| `error_url` | string | Error redirect |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Form Types
| Type | Description |
|------|-------------|
| `landing` | Public lead capture forms |
| `survey` | Feedback and data collection |
| `appointment` | Scheduling with confirm/cancel |
| `subscription` | Newsletter subscriptions |
| `referral` | Referral tracking |
| `app_form` | Dynamic business forms with schemas |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "400",
    "source": "Form Creation",
    "code": "frm_10110",
    "title": "Invalid Form Domain",
    "detail": "The form domain must be the same domain or subdomain of the company preferred domain."
  }]
}
```
