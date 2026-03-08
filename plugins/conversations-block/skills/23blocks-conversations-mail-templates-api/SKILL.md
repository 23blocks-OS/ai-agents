---
name: 23blocks-conversations-mail-templates-api
description: Manage email templates via Mandrill integration. Use when creating, updating, publishing, or monitoring transactional email templates for the Conversations Block.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Mail Templates API

Manage email templates through Mandrill integration. Create, update, publish, and monitor template statistics for transactional email delivery within the Conversations Block.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Conversations API base URL | `https://conversations.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

```bash
curl -X GET "$BLOCKS_API_URL/mailtemplates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/mailtemplates` | List mail templates |
| POST | `/mailtemplates` | Create mail template |
| PUT | `/mailtemplates` | Update mail template |
| POST | `/mailtemplates/mandrill/create` | Create Mandrill template |
| POST | `/mailtemplates/mandrill/update` | Update Mandrill template |
| POST | `/mailtemplates/mandrill/publish` | Publish Mandrill template |
| GET | `/mailtemplates/mandrill/stats` | Get Mandrill template stats |

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

## Error Response Format

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

Common status codes: `401` Unauthorized, `404` Not Found, `409` Conflict, `422` Unprocessable Entity.
