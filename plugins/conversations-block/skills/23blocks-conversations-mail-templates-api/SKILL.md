---
name: 23blocks-conversations-mail-templates-api
description: "Conversations Block Mandrill email templates: create, update, publish, stats. Use for chat-related transactional email."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Mail Templates API

Manage email templates through Mandrill integration. Create, update, publish, and monitor template statistics for transactional email delivery within the Conversations Block.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://realtime.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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
