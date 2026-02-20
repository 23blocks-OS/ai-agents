---
name: onboarding-mail-templates-api
description: Manage 23blocks onboarding email templates via REST API. Use when creating, updating, or listing email templates, managing Mandrill templates, publishing templates, or viewing delivery statistics.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Mail Templates API

Complete API reference for 23blocks onboarding email template management with Mandrill integration.

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
curl -X GET "$BLOCKS_API_URL/mailtemplates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /mailtemplates/ - List Templates

Lists all email templates.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mailtemplates?page=1&records=20" \
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
      "id": "template-uuid-123",
      "type": "mail_template",
      "attributes": {
        "unique_id": "template-uuid-123",
        "name": "Onboarding Welcome",
        "template_name": "onboarding-welcome",
        "from_email": "noreply@example.com",
        "from_name": "Acme Corp",
        "from_subject": "Welcome to Acme!",
        "provider": "mandrill",
        "status": "published",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 8
  }
}
```

---

### GET /mailtemplates/:unique_id - Get Template

Retrieves a single email template by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mailtemplates/template-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "template-uuid-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "template-uuid-123",
      "name": "Onboarding Welcome",
      "template_name": "onboarding-welcome",
      "template_html": "<h1>Welcome {{name}}</h1><p>Start your onboarding journey.</p>",
      "template_text": "Welcome {{name}}. Start your onboarding journey.",
      "from_email": "noreply@example.com",
      "from_name": "Acme Corp",
      "from_subject": "Welcome to Acme!",
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

### POST /mailtemplates - Create Template

Creates a new email template.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mailtemplates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mail_template": {
      "name": "Journey Reminder",
      "template_name": "journey-reminder",
      "from_email": "noreply@example.com",
      "from_name": "Acme Corp",
      "from_subject": "Continue Your Onboarding",
      "template_html": "<h1>Hi {{name}}</h1><p>You are on step {{current_step}} of {{total_steps}}. <a href=\"{{resume_url}}\">Continue now</a>.</p>",
      "template_text": "Hi {{name}}, you are on step {{current_step}} of {{total_steps}}. Continue at {{resume_url}}"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Template display name |
| `template_name` | string | Yes | Template slug/identifier |
| `from_email` | string | Yes | Sender email address |
| `from_name` | string | No | Sender display name |
| `from_subject` | string | Yes | Email subject line |
| `template_html` | string | No | HTML template content |
| `template_text` | string | No | Plain text template content |

**Response 201:**
```json
{
  "data": {
    "id": "new-template-uuid",
    "type": "mail_template",
    "attributes": {
      "unique_id": "new-template-uuid",
      "name": "Journey Reminder",
      "template_name": "journey-reminder",
      "status": "draft",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /mailtemplates/:unique_id - Update Template

Updates an existing email template.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/mailtemplates/template-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mail_template": {
      "from_subject": "Don'\''t Miss Out - Continue Your Setup",
      "template_html": "<h1>Hi {{name}}</h1><p>Pick up where you left off.</p>"
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
      "from_subject": "Don't Miss Out - Continue Your Setup",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Mandrill Integration

### POST /mailtemplates/:unique_id/mandrill - Create Mandrill Template

Creates the template in Mandrill from the local template content.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mailtemplates/template-uuid-123/mandrill" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "template-uuid-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "template-uuid-123",
      "mandrill_slug": "onboarding-welcome",
      "mandrill_status": "created"
    }
  }
}
```

---

### PUT /mailtemplates/:unique_id/mandrill - Update Mandrill Template

Updates the template in Mandrill with latest local content.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/mailtemplates/template-uuid-123/mandrill" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "template-uuid-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "template-uuid-123",
      "mandrill_status": "updated"
    }
  }
}
```

---

### PUT /mailtemplates/:unique_id/mandrill/publish - Publish Mandrill Template

Publishes the template in Mandrill, making it available for sending.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/mailtemplates/template-uuid-123/mandrill/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "template-uuid-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "template-uuid-123",
      "status": "published",
      "mandrill_status": "published",
      "published_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### GET /mailtemplates/:unique_id/mandrill/stats - Get Mandrill Stats

Retrieves delivery statistics for the template from Mandrill.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mailtemplates/template-uuid-123/mandrill/stats" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "template-uuid-123",
    "type": "mail_template_stats",
    "attributes": {
      "sends": 1520,
      "opens": 890,
      "clicks": 342,
      "hard_bounces": 5,
      "soft_bounces": 12,
      "rejects": 2,
      "complaints": 0,
      "unique_opens": 780,
      "unique_clicks": 295
    }
  }
}
```

---

## Data Models

### MailTemplate
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Template display name |
| `template_name` | string | Template slug/identifier |
| `template_html` | string | HTML content with variables |
| `template_text` | string | Plain text content |
| `from_email` | string | Sender email |
| `from_name` | string | Sender display name |
| `from_subject` | string | Email subject line |
| `provider` | string | Email provider (mandrill) |
| `status` | string | draft, published |
| `mandrill_slug` | string | Mandrill template identifier |
| `mandrill_status` | string | Mandrill sync status |
| `published_at` | timestamp | Publication time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### MailTemplateStats
| Field | Type | Description |
|-------|------|-------------|
| `sends` | integer | Total emails sent |
| `opens` | integer | Total opens |
| `clicks` | integer | Total clicks |
| `hard_bounces` | integer | Hard bounce count |
| `soft_bounces` | integer | Soft bounce count |
| `rejects` | integer | Rejected count |
| `complaints` | integer | Spam complaints |
| `unique_opens` | integer | Unique open count |
| `unique_clicks` | integer | Unique click count |

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
// MailTemplatesService — client.onboarding.mailTemplates
client.onboarding.mailTemplates.list(params?: ListMailTemplatesParams): Promise<PageResult<MailTemplate>>;
client.onboarding.mailTemplates.get(uniqueId: string): Promise<MailTemplate>;
client.onboarding.mailTemplates.create(data: CreateMailTemplateRequest): Promise<MailTemplate>;
client.onboarding.mailTemplates.update(uniqueId: string, data: UpdateMailTemplateRequest): Promise<MailTemplate>;
client.onboarding.mailTemplates.getMandrillStats(uniqueId: string): Promise<MandrillTemplateStats>;
client.onboarding.mailTemplates.createMandrillTemplate(uniqueId: string, data: CreateMandrillTemplateRequest): Promise<MailTemplate>;
client.onboarding.mailTemplates.updateMandrillTemplate(uniqueId: string, data: UpdateMandrillTemplateRequest): Promise<MailTemplate>;
client.onboarding.mailTemplates.publishMandrill(uniqueId: string): Promise<MailTemplate>;
```

### TypeScript Types

```typescript
import type {
  MailTemplate,
  CreateMailTemplateRequest,
  UpdateMailTemplateRequest,
  ListMailTemplatesParams,
  CreateMandrillTemplateRequest,
  UpdateMandrillTemplateRequest,
  MandrillTemplateStats,
} from '@23blocks/block-onboarding';
```

### React Hook

```typescript
import { useOnboardingBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useOnboardingBlock();
  const result = await client.onboarding.mailTemplates.list({ page: 1, perPage: 20 });
}
```
