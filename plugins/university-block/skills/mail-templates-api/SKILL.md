---
name: university-mail-templates-api
description: Manage 23blocks university email templates via REST API. Use for configuring Mandrill templates, publishing to provider, viewing statistics, and customizing university notification emails.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Mail Templates API

Complete API reference for 23blocks university email template management with Mandrill integration.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://university.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | University API base URL | `https://university.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/mailtemplates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /mailtemplates/ - List Mail Templates

Lists all mail templates.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mailtemplates?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "tpl-uuid-123",
      "type": "mail_template",
      "attributes": {
        "unique_id": "tpl-uuid-123",
        "name": "Course Enrollment Confirmation",
        "template_name": "course-enrollment-v1",
        "template_html": "<html>...",
        "template_text": "You have been enrolled in {{course_name}}...",
        "from_email": "university@example.com",
        "from_name": "University Admin",
        "from_subject": "Enrollment Confirmed - {{course_name}}",
        "provider": "mandrill",
        "status": "published",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 28
  }
}
```

---

### GET /mailtemplates/:unique_id - Get Mail Template

Retrieves a specific mail template.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mailtemplates/tpl-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "tpl-uuid-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "tpl-uuid-123",
      "name": "Course Enrollment Confirmation",
      "template_name": "course-enrollment-v1",
      "template_html": "<!DOCTYPE html><html><body><h1>Welcome to {{course_name}}</h1><p>Dear {{student_name}}, your enrollment has been confirmed.</p></body></html>",
      "template_text": "Welcome to {{course_name}}. Dear {{student_name}}, your enrollment has been confirmed.",
      "from_email": "university@example.com",
      "from_name": "University Admin",
      "from_subject": "Enrollment Confirmed - {{course_name}}",
      "provider": "mandrill",
      "status": "published",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Template not found

---

### POST /mailtemplates - Create Mail Template

Creates a new mail template.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mailtemplates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mail_template": {
      "name": "Session Reminder",
      "template_name": "session-reminder-v1",
      "template_html": "<!DOCTYPE html><html><body><h1>Upcoming Session</h1><p>Dear {{student_name}}, you have a coaching session on {{session_date}} at {{session_time}}.</p></body></html>",
      "template_text": "Dear {{student_name}}, you have a coaching session on {{session_date}} at {{session_time}}.",
      "from_email": "reminders@university.com",
      "from_name": "University Reminders",
      "from_subject": "Coaching Session Reminder - {{session_date}}",
      "provider": "mandrill",
      "status": "draft"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Template display name |
| `template_name` | string | Yes | Provider template identifier |
| `template_html` | text | Yes | HTML content |
| `template_text` | string | No | Plain text content |
| `from_email` | string | Yes | Sender email |
| `from_name` | string | No | Sender display name |
| `from_subject` | string | Yes | Email subject |
| `provider` | string | No | Email provider (mandrill) |
| `status` | enum | No | draft, published (default: draft) |

**Response 201:**
```json
{
  "data": {
    "id": "tpl-uuid-new",
    "type": "mail_template",
    "attributes": {
      "unique_id": "tpl-uuid-new",
      "name": "Session Reminder",
      "template_name": "session-reminder-v1",
      "from_email": "reminders@university.com",
      "from_subject": "Coaching Session Reminder - {{session_date}}",
      "provider": "mandrill",
      "status": "draft",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /mailtemplates/:unique_id - Update Mail Template

Updates an existing mail template.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/mailtemplates/tpl-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mail_template": {
      "from_subject": "Updated: Enrollment Confirmed - {{course_name}}",
      "template_html": "<!DOCTYPE html><html>...updated content...</html>"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "tpl-uuid-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "tpl-uuid-123",
      "name": "Course Enrollment Confirmation",
      "from_subject": "Updated: Enrollment Confirmed - {{course_name}}",
      "status": "draft",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

## Mandrill Integration

### POST /mailtemplates/:unique_id/mandrill - Create in Mandrill

Creates the template in Mandrill provider.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mailtemplates/tpl-uuid-123/mandrill" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "tpl-uuid-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "tpl-uuid-123",
      "name": "Course Enrollment Confirmation",
      "provider": "mandrill",
      "status": "draft",
      "mandrill_slug": "course-enrollment-v1"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Mandrill API error
- `404 Not Found` - Template not found

---

### PUT /mailtemplates/:unique_id/mandrill - Update in Mandrill

Updates the template in Mandrill provider.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/mailtemplates/tpl-uuid-123/mandrill" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "tpl-uuid-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "tpl-uuid-123",
      "name": "Course Enrollment Confirmation",
      "provider": "mandrill",
      "status": "draft"
    }
  }
}
```

---

### PUT /mailtemplates/:unique_id/mandrill/publish - Publish in Mandrill

Publishes the template in Mandrill, making it available for sending.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/mailtemplates/tpl-uuid-123/mandrill/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "tpl-uuid-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "tpl-uuid-123",
      "name": "Course Enrollment Confirmation",
      "provider": "mandrill",
      "status": "published"
    }
  }
}
```

---

### GET /mailtemplates/:unique_id/mandrill/stats - Template Statistics

Retrieves Mandrill statistics for a template.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mailtemplates/tpl-uuid-123/mandrill/stats" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "template_id": "tpl-uuid-123",
    "stats": {
      "sent": 2500,
      "delivered": 2480,
      "opens": 1890,
      "unique_opens": 1650,
      "clicks": 445,
      "unique_clicks": 380,
      "bounces": 20,
      "soft_bounces": 15,
      "hard_bounces": 5,
      "spam_complaints": 1,
      "unsubscribes": 3,
      "open_rate": 0.665,
      "click_rate": 0.153,
      "bounce_rate": 0.008
    },
    "last_30_days": {
      "sent": 600,
      "delivered": 598,
      "opens": 420
    }
  }
}
```

---

## Template Variables

Templates use Handlebars syntax for variable interpolation:

### Common Variables
| Variable | Description |
|----------|-------------|
| `{{student_name}}` | Student name |
| `{{teacher_name}}` | Teacher/coach name |
| `{{course_name}}` | Course name |
| `{{session_date}}` | Session date |
| `{{session_time}}` | Session time |
| `{{location}}` | Session or event location |
| `{{token}}` | Registration token |
| `{{company_name}}` | Institution name |

---

## Data Models

### MailTemplate
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Template display name |
| `template_name` | string | Provider template identifier |
| `template_html` | text | HTML content |
| `template_text` | text | Plain text content |
| `from_email` | string | Sender email |
| `from_name` | string | Sender display name |
| `from_subject` | string | Email subject |
| `provider` | string | Email provider (mandrill) |
| `status` | enum | draft, published |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "mandrill_error",
    "title": "Mandrill API Error",
    "detail": "Template could not be created in Mandrill."
  }]
}
```
