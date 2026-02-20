---
name: conversations-mail-templates-api
description: Manage email templates through Mandrill integration in the 23blocks Conversations Block
version: 1.0.0
tags:
  - conversations
  - email
  - templates
  - mandrill
env:
  - BLOCKS_API_URL
  - BLOCKS_AUTH_TOKEN
  - BLOCKS_API_KEY
---

# Mail Templates API

Manage email templates through Mandrill integration. Create, update, publish, and monitor template statistics for transactional email delivery within the Conversations Block.

## Authentication

All requests require the following headers:

```
Authorization: Bearer $BLOCKS_AUTH_TOKEN
AppId: $BLOCKS_API_KEY
```

## Base URL

```
https://conversations.api.us.23blocks.com
```

---

## Endpoints

### List Mail Templates

Retrieve all email templates.

**Endpoint:** `GET /mailtemplates`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number for pagination |
| per_page | integer | No | Results per page (default: 25) |
| status | string | No | Filter by status: `draft`, `published` |
| search | string | No | Search by template name |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/mailtemplates?page=1&per_page=25" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "mt_001",
      "type": "mail_templates",
      "attributes": {
        "unique_id": "mt_001",
        "name": "welcome-email",
        "slug": "welcome-email",
        "subject": "Welcome to {{company_name}}",
        "from_email": "noreply@example.com",
        "from_name": "Example App",
        "html_content": "<html><body><h1>Welcome, {{name}}!</h1></body></html>",
        "text_content": "Welcome, {{name}}!",
        "status": "published",
        "mandrill_slug": "welcome-email",
        "labels": ["onboarding", "welcome"],
        "created_at": "2025-01-05T08:00:00Z",
        "updated_at": "2025-01-10T12:00:00Z"
      }
    }
  ],
  "meta": {
    "total": 12,
    "page": 1,
    "per_page": 25
  }
}
```

---

### Create Mail Template

Create a new email template.

**Endpoint:** `POST /mailtemplates`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Template name (used as identifier) |
| subject | string | Yes | Email subject line (supports merge tags) |
| from_email | string | Yes | Sender email address |
| from_name | string | No | Sender display name |
| html_content | string | Yes | HTML email body (supports merge tags) |
| text_content | string | No | Plain text email body |
| labels | array | No | Array of label strings for categorization |
| metadata | object | No | Additional metadata |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/mailtemplates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "invite-notification",
    "subject": "You have been invited to {{group_name}}",
    "from_email": "noreply@example.com",
    "from_name": "Example App",
    "html_content": "<html><body><h1>Hello {{name}},</h1><p>You have been invited to join {{group_name}}.</p><a href=\"{{invite_url}}\">Accept Invite</a></body></html>",
    "text_content": "Hello {{name}}, You have been invited to join {{group_name}}. Click here: {{invite_url}}",
    "labels": ["invites", "notifications"]
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "mt_new001",
    "type": "mail_templates",
    "attributes": {
      "unique_id": "mt_new001",
      "name": "invite-notification",
      "slug": "invite-notification",
      "subject": "You have been invited to {{group_name}}",
      "from_email": "noreply@example.com",
      "from_name": "Example App",
      "html_content": "<html><body><h1>Hello {{name}},</h1><p>You have been invited to join {{group_name}}.</p><a href=\"{{invite_url}}\">Accept Invite</a></body></html>",
      "text_content": "Hello {{name}}, You have been invited to join {{group_name}}. Click here: {{invite_url}}",
      "status": "draft",
      "mandrill_slug": null,
      "labels": ["invites", "notifications"],
      "created_at": "2025-01-15T11:00:00Z",
      "updated_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

### Update Mail Template

Update an existing email template.

**Endpoint:** `PUT /mailtemplates`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| unique_id | string | Yes | The template unique ID |
| name | string | No | Updated template name |
| subject | string | No | Updated subject line |
| from_email | string | No | Updated sender email |
| from_name | string | No | Updated sender name |
| html_content | string | No | Updated HTML content |
| text_content | string | No | Updated text content |
| labels | array | No | Updated labels |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/mailtemplates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "unique_id": "mt_new001",
    "subject": "{{inviter_name}} invited you to {{group_name}}",
    "html_content": "<html><body><h1>Hello {{name}},</h1><p>{{inviter_name}} has invited you to join {{group_name}}.</p><a href=\"{{invite_url}}\">Accept Invite</a></body></html>"
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "mt_new001",
    "type": "mail_templates",
    "attributes": {
      "unique_id": "mt_new001",
      "name": "invite-notification",
      "subject": "{{inviter_name}} invited you to {{group_name}}",
      "status": "draft",
      "updated_at": "2025-01-15T12:00:00Z"
    }
  }
}
```

---

## Mandrill Integration

### Create Mandrill Template

Sync a local template to Mandrill.

**Endpoint:** `POST /mailtemplates/mandrill/create`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| template_unique_id | string | Yes | Local template unique ID |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/mailtemplates/mandrill/create" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template_unique_id": "mt_new001"
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "mt_new001",
    "type": "mail_templates",
    "attributes": {
      "unique_id": "mt_new001",
      "name": "invite-notification",
      "mandrill_slug": "invite-notification",
      "mandrill_synced": true,
      "mandrill_synced_at": "2025-01-15T12:30:00Z"
    }
  }
}
```

---

### Update Mandrill Template

Push local template changes to Mandrill.

**Endpoint:** `POST /mailtemplates/mandrill/update`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| template_unique_id | string | Yes | Local template unique ID |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/mailtemplates/mandrill/update" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template_unique_id": "mt_new001"
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "mt_new001",
    "type": "mail_templates",
    "attributes": {
      "unique_id": "mt_new001",
      "mandrill_slug": "invite-notification",
      "mandrill_synced": true,
      "mandrill_synced_at": "2025-01-15T13:00:00Z"
    }
  }
}
```

---

### Publish Mandrill Template

Publish a template on Mandrill, making it available for sending.

**Endpoint:** `POST /mailtemplates/mandrill/publish`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| template_unique_id | string | Yes | Local template unique ID |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/mailtemplates/mandrill/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template_unique_id": "mt_new001"
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "mt_new001",
    "type": "mail_templates",
    "attributes": {
      "unique_id": "mt_new001",
      "name": "invite-notification",
      "mandrill_slug": "invite-notification",
      "status": "published",
      "published_at": "2025-01-15T13:30:00Z",
      "updated_at": "2025-01-15T13:30:00Z"
    }
  }
}
```

---

### Get Mandrill Template Stats

Retrieve sending statistics for a Mandrill template.

**Endpoint:** `GET /mailtemplates/mandrill/stats`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| template_unique_id | string | Yes | Local template unique ID |
| start_date | string | No | ISO 8601 start date for stats range |
| end_date | string | No | ISO 8601 end date for stats range |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/mailtemplates/mandrill/stats?template_unique_id=mt_001&start_date=2025-01-01T00:00:00Z&end_date=2025-01-15T23:59:59Z" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "mt_001",
    "type": "mail_template_stats",
    "attributes": {
      "template_unique_id": "mt_001",
      "name": "welcome-email",
      "mandrill_slug": "welcome-email",
      "stats": {
        "sent": 1250,
        "delivered": 1230,
        "opens": 890,
        "unique_opens": 780,
        "clicks": 450,
        "unique_clicks": 380,
        "bounces": {
          "hard": 5,
          "soft": 15
        },
        "complaints": 2,
        "unsubs": 8,
        "rejects": 0
      },
      "rates": {
        "delivery_rate": 98.4,
        "open_rate": 71.2,
        "click_rate": 36.0,
        "bounce_rate": 1.6,
        "complaint_rate": 0.16
      },
      "period": {
        "start_date": "2025-01-01T00:00:00Z",
        "end_date": "2025-01-15T23:59:59Z"
      }
    }
  }
}
```

---

## Data Model

### Mail Template

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the template |
| name | string | Template name |
| slug | string | URL-friendly template identifier |
| subject | string | Email subject line (supports merge tags like `{{name}}`) |
| from_email | string | Sender email address |
| from_name | string | Sender display name |
| html_content | string | HTML email body (supports merge tags) |
| text_content | string | Plain text email body |
| status | string | Template status: `draft`, `published` |
| mandrill_slug | string | Mandrill template slug (null if not synced) |
| mandrill_synced | boolean | Whether template is synced to Mandrill |
| mandrill_synced_at | datetime | Last Mandrill sync timestamp |
| published_at | datetime | Publication timestamp |
| labels | array | Categorization labels |
| metadata | object | Arbitrary key-value metadata |
| created_at | datetime | Template creation timestamp |
| updated_at | datetime | Last update timestamp |

### Template Stats

| Field | Type | Description |
|-------|------|-------------|
| sent | integer | Total emails sent |
| delivered | integer | Successfully delivered emails |
| opens | integer | Total opens (including repeats) |
| unique_opens | integer | Unique opens |
| clicks | integer | Total link clicks |
| unique_clicks | integer | Unique link clicks |
| bounces | object | Hard and soft bounce counts |
| complaints | integer | Spam complaints |
| unsubs | integer | Unsubscribes |
| rejects | integer | Rejected sends |

---

## Error Responses

### 401 Unauthorized

```json
{
  "errors": [
    {
      "status": "401",
      "title": "Unauthorized",
      "detail": "Invalid or missing authentication token"
    }
  ]
}
```

### 404 Not Found

```json
{
  "errors": [
    {
      "status": "404",
      "title": "Not Found",
      "detail": "Template with unique_id 'mt_invalid' not found"
    }
  ]
}
```

### 409 Conflict

```json
{
  "errors": [
    {
      "status": "409",
      "title": "Conflict",
      "detail": "Template with name 'welcome-email' already exists in Mandrill"
    }
  ]
}
```

### 422 Unprocessable Entity

```json
{
  "errors": [
    {
      "status": "422",
      "title": "Unprocessable Entity",
      "detail": "Template must be synced to Mandrill before publishing"
    }
  ]
}
```
