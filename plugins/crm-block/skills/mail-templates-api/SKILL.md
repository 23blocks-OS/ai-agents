---
name: crm-mail-templates-api
description: Manage 23blocks CRM mail templates via REST API. Use when creating, updating, or managing email templates and integrating with Mandrill for email delivery, publishing, and statistics.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# CRM Mail Templates API

Complete API reference for 23blocks CRM mail template management with Mandrill integration for email delivery and statistics.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://crm.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
echo "All credentials configured"
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | CRM API base URL | `https://crm.api.us.23blocks.com` |
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

## Standard Template Endpoints

### GET /mailtemplates - List Mail Templates

Lists all mail templates.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mailtemplates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "template-uuid-123",
      "type": "mail_template",
      "attributes": {
        "unique_id": "template-uuid-123",
        "name": "Welcome Email",
        "slug": "welcome-email",
        "subject": "Welcome to {{company_name}}!",
        "from_email": "noreply@example.com",
        "from_name": "Support Team",
        "html_content": "<h1>Welcome!</h1><p>Thank you for joining us.</p>",
        "text_content": "Welcome! Thank you for joining us.",
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalRecords": 12
  }
}
```

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
      "name": "Meeting Reminder",
      "slug": "meeting-reminder",
      "subject": "Reminder: {{meeting_title}} on {{meeting_date}}",
      "from_email": "noreply@example.com",
      "from_name": "Calendar Bot",
      "html_content": "<h1>Meeting Reminder</h1><p>You have a meeting: {{meeting_title}}</p>",
      "text_content": "Meeting Reminder: You have a meeting: {{meeting_title}}"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Template name |
| `slug` | string | Yes | URL-safe identifier |
| `subject` | string | Yes | Email subject (supports merge tags) |
| `from_email` | string | No | Sender email address |
| `from_name` | string | No | Sender name |
| `html_content` | string | Yes | HTML email body |
| `text_content` | string | No | Plain text email body |

**Response 201:**
```json
{
  "data": {
    "id": "template-uuid-456",
    "type": "mail_template",
    "attributes": {
      "unique_id": "template-uuid-456",
      "name": "Meeting Reminder",
      "slug": "meeting-reminder",
      "subject": "Reminder: {{meeting_title}} on {{meeting_date}}",
      "status": "draft",
      "created_at": "2026-02-15T10:30:00Z"
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
curl -X PUT "$BLOCKS_API_URL/mailtemplates/template-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mail_template": {
      "subject": "Welcome to {{company_name}} - Get Started!",
      "html_content": "<h1>Welcome!</h1><p>Here are your next steps...</p>"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "template-uuid-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "template-uuid-123",
      "subject": "Welcome to {{company_name}} - Get Started!",
      "updated_at": "2026-02-15T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Template not found

---

## Mandrill Integration

### POST /mailtemplates/mandrill/create - Create Mandrill Template

Creates a template in Mandrill for transactional email delivery.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mailtemplates/mandrill/create" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "name": "welcome-email",
      "from_email": "noreply@example.com",
      "from_name": "Support Team",
      "subject": "Welcome to {{company_name}}!",
      "code": "<h1>Welcome!</h1><p>Thank you for joining us, *|FNAME|*.</p>"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Template slug/name |
| `from_email` | string | Yes | Sender email |
| `from_name` | string | No | Sender name |
| `subject` | string | Yes | Email subject |
| `code` | string | Yes | HTML template code with Mandrill merge tags |

**Response 201:**
```json
{
  "data": {
    "slug": "welcome-email",
    "name": "welcome-email",
    "status": "draft",
    "created_at": "2026-02-15T10:30:00Z"
  }
}
```

---

### PUT /mailtemplates/mandrill/update - Update Mandrill Template

Updates a template in Mandrill.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/mailtemplates/mandrill/update" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "name": "welcome-email",
      "code": "<h1>Welcome!</h1><p>Updated content for *|FNAME|*.</p>"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "slug": "welcome-email",
    "status": "draft",
    "updated_at": "2026-02-15T14:00:00Z"
  }
}
```

---

### PUT /mailtemplates/mandrill/publish - Publish Mandrill Template

Publishes a Mandrill template, making it available for sending.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/mailtemplates/mandrill/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "name": "welcome-email"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "slug": "welcome-email",
    "status": "published",
    "published_at": "2026-02-15T14:30:00Z"
  }
}
```

---

### GET /mailtemplates/mandrill/stats - Mandrill Template Stats

Retrieves sending statistics for Mandrill templates.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mailtemplates/mandrill/stats" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "slug": "welcome-email",
      "name": "Welcome Email",
      "stats": {
        "sent": 1250,
        "opens": 980,
        "clicks": 345,
        "bounces": 12,
        "complaints": 2,
        "open_rate": 78.4,
        "click_rate": 27.6
      }
    },
    {
      "slug": "meeting-reminder",
      "name": "Meeting Reminder",
      "stats": {
        "sent": 850,
        "opens": 720,
        "clicks": 190,
        "bounces": 5,
        "complaints": 0,
        "open_rate": 84.7,
        "click_rate": 22.4
      }
    }
  ]
}
```

---

## Data Models

### MailTemplate
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Template name |
| `slug` | string | URL-safe identifier |
| `subject` | string | Email subject line |
| `from_email` | string | Sender email address |
| `from_name` | string | Sender display name |
| `html_content` | string | HTML email body |
| `text_content` | string | Plain text email body |
| `status` | enum | draft, active, archived |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Template Not Found",
    "detail": "The requested mail template could not be found."
  }]
}
```
