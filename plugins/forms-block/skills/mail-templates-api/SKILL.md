---
name: mail-templates-api
description: Manage 23blocks email templates via REST API. Use for configuring Mandrill/SendGrid templates, viewing statistics, and customizing form emails.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Mail Templates API

Complete API reference for 23blocks email template management with Mandrill/SendGrid integration.

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

### GET /mailtemplates - List Mail Templates

Lists all mail templates.

**Request:**
```bash
curl -X GET "$API_URL/mailtemplates?page=1&records=20" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "tpl-123",
      "type": "mail_template",
      "attributes": {
        "unique_id": "tpl-123",
        "name": "Welcome Email",
        "event_name": "user_welcome",
        "from_email": "hello@company.com",
        "from_name": "Company Name",
        "from_subject": "Welcome to Company!",
        "template_provider": "mandrill",
        "template_slug": "welcome-email-v2",
        "template_html": "<html>...",
        "template_text": "Welcome {{name}}...",
        "preferred_language": "en",
        "enabled": true,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 42
  }
}
```

---

### GET /mailtemplates/:unique_id - Get Mail Template

Retrieves a specific mail template.

**Request:**
```bash
curl -X GET "$API_URL/mailtemplates/tpl-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 200:**
```json
{
  "data": {
    "id": "tpl-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "tpl-123",
      "name": "OTP Verification",
      "event_name": "otp_verification_code",
      "from_email": "security@company.com",
      "from_name": "Company Security",
      "from_subject": "Your Verification Code",
      "template_provider": "mandrill",
      "template_slug": "otp-verification",
      "template_html": "<!DOCTYPE html><html>...",
      "template_text": "Your verification code is: {{code}}",
      "preferred_language": "en",
      "enabled": true,
      "merge_language": "handlebars",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### POST /mailtemplates - Create Mail Template

Creates a new mail template.

**Request:**
```bash
curl -X POST "$API_URL/mailtemplates" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "mail_template": {
      "name": "Form Confirmation",
      "event_name": "form_confirmation",
      "from_email": "forms@company.com",
      "from_name": "Company Forms",
      "from_subject": "Thank you for your submission",
      "template_provider": "mandrill",
      "template_slug": "form-confirmation-v1",
      "template_html": "<!DOCTYPE html><html><body><h1>Thank you, {{name}}!</h1><p>We received your form submission.</p></body></html>",
      "template_text": "Thank you, {{name}}! We received your form submission.",
      "preferred_language": "en",
      "enabled": true
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Template display name |
| `event_name` | string | Yes | Event trigger name |
| `from_email` | string | Yes | Sender email |
| `from_name` | string | No | Sender display name |
| `from_subject` | string | Yes | Email subject |
| `template_provider` | enum | No | mandrill, sendgrid, action_mailer |
| `template_slug` | string | No | Provider template ID |
| `template_html` | string | Yes | HTML content |
| `template_text` | string | No | Plain text content |
| `preferred_language` | string | No | Language code (en, es) |
| `enabled` | boolean | No | Template active status |

**Response 201:**
```json
{
  "data": {
    "id": "new-tpl-id",
    "type": "mail_template",
    "attributes": {
      "unique_id": "new-tpl-id",
      "name": "Form Confirmation",
      "event_name": "form_confirmation",
      "enabled": true,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /mailtemplates/:unique_id - Update Mail Template

Updates an existing mail template.

**Request:**
```bash
curl -X PUT "$API_URL/mailtemplates/tpl-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "mail_template": {
      "from_subject": "Updated Subject Line",
      "template_html": "<!DOCTYPE html><html>...updated...</html>",
      "enabled": true
    }
  }'
```

---

### DELETE /mailtemplates/:unique_id - Delete Mail Template

Deletes a mail template.

**Request:**
```bash
curl -X DELETE "$API_URL/mailtemplates/tpl-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 204:** No content

---

### GET /mailtemplates/:unique_id/stats - Get Template Statistics

Retrieves Mandrill statistics for a template.

**Request:**
```bash
curl -X GET "$API_URL/mailtemplates/tpl-123/stats" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 200:**
```json
{
  "data": {
    "template_id": "tpl-123",
    "stats": {
      "sent": 1500,
      "delivered": 1485,
      "opens": 890,
      "unique_opens": 750,
      "clicks": 245,
      "unique_clicks": 200,
      "bounces": 15,
      "soft_bounces": 10,
      "hard_bounces": 5,
      "spam_complaints": 2,
      "unsubscribes": 8,
      "open_rate": 0.504,
      "click_rate": 0.135,
      "bounce_rate": 0.01
    },
    "last_30_days": {
      "sent": 450,
      "delivered": 448,
      "opens": 280
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
| `{{name}}` | Recipient name |
| `{{email}}` | Recipient email |
| `{{form_name}}` | Form name |
| `{{company_name}}` | Company name |
| `{{magic_link}}` | Magic link URL |
| `{{verification_code}}` | OTP code |
| `{{expires_in_minutes}}` | Expiration time |

### OTP Template Example
```html
<!DOCTYPE html>
<html>
<head>
  <title>Verification Code</title>
</head>
<body>
  <h1>Your Verification Code</h1>
  <p>Hello {{name}},</p>
  <p>Your verification code for {{form_name}} is:</p>
  <div style="font-size: 32px; font-weight: bold; padding: 20px;">
    {{verification_code}}
  </div>
  <p>This code expires in {{expires_in_minutes}} minutes.</p>
  <p>If you didn't request this code, please ignore this email.</p>
</body>
</html>
```

---

## Data Models

### Mail Template
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Template display name |
| `event_name` | string | Event trigger name |
| `from_email` | string | Sender email |
| `from_name` | string | Sender display name |
| `from_subject` | string | Email subject |
| `template_provider` | enum | mandrill, sendgrid, action_mailer |
| `template_slug` | string | Provider template ID |
| `template_html` | text | HTML content |
| `template_text` | text | Plain text content |
| `preferred_language` | string | Language code |
| `enabled` | boolean | Active status |
| `merge_language` | string | handlebars, mailchimp |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Event Names
| Event | Description |
|-------|-------------|
| `user_welcome` | New user welcome |
| `form_confirmation` | Form submission confirmation |
| `otp_verification_code` | OTP email |
| `magic_link` | Magic link access |
| `appointment_confirmation` | Appointment confirmed |
| `appointment_reminder` | Appointment reminder |
| `appointment_cancellation` | Appointment cancelled |
| `survey_invitation` | Survey invitation |

---

## Error Codes

| HTTP Status | Code | Description |
|-------------|------|-------------|
| 404 | Not Found | Template not found |
| 422 | Unprocessable Entity | Validation error |
| 400 | Bad Request | Invalid parameters |
