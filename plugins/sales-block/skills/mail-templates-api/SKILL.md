---
name: sales-mail-templates-api
description: Manage 23blocks Sales email templates via REST API. Use for configuring Mandrill templates, viewing email statistics, and customizing sales notification emails.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Mail Templates API

Complete API reference for 23blocks Sales email template management with Mandrill integration.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://sales.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Sales API base URL | `https://sales.api.us.23blocks.com` |
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

### GET /mailtemplates - List Mail Templates

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
      "id": "tpl-123",
      "type": "mail_template",
      "attributes": {
        "unique_id": "tpl-123",
        "name": "Order Confirmation",
        "event_name": "order_confirmed",
        "from_email": "orders@company.com",
        "from_name": "Company Orders",
        "from_subject": "Your Order #{{order_id}} is Confirmed",
        "template_provider": "mandrill",
        "template_slug": "order-confirmation-v2",
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
curl -X GET "$BLOCKS_API_URL/mailtemplates/tpl-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "tpl-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "tpl-123",
      "name": "Order Confirmation",
      "event_name": "order_confirmed",
      "from_email": "orders@company.com",
      "from_name": "Company Orders",
      "from_subject": "Your Order #{{order_id}} is Confirmed",
      "template_provider": "mandrill",
      "template_slug": "order-confirmation-v2",
      "template_html": "<!DOCTYPE html><html>...",
      "template_text": "Your order {{order_id}} has been confirmed.",
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
curl -X POST "$BLOCKS_API_URL/mailtemplates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mail_template": {
      "name": "Payment Receipt",
      "event_name": "payment_confirmed",
      "from_email": "billing@company.com",
      "from_name": "Company Billing",
      "from_subject": "Payment Receipt - {{amount}}",
      "template_provider": "mandrill",
      "template_slug": "payment-receipt-v1",
      "template_html": "<!DOCTYPE html><html><body><h1>Payment Received</h1><p>Thank you, {{name}}! We received your payment of {{amount}}.</p></body></html>",
      "template_text": "Thank you, {{name}}! We received your payment of {{amount}}.",
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
      "name": "Payment Receipt",
      "event_name": "payment_confirmed",
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
curl -X PUT "$BLOCKS_API_URL/mailtemplates/tpl-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mail_template": {
      "from_subject": "Updated: Your Order #{{order_id}} is Confirmed",
      "template_html": "<!DOCTYPE html><html>...updated...</html>",
      "enabled": true
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "tpl-123",
    "type": "mail_template",
    "attributes": {
      "unique_id": "tpl-123",
      "from_subject": "Updated: Your Order #{{order_id}} is Confirmed",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /mailtemplates/:unique_id - Delete Mail Template

Deletes a mail template.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/mailtemplates/tpl-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### GET /mailtemplates/:unique_id/stats - Get Template Statistics

Retrieves Mandrill statistics for a template.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mailtemplates/tpl-123/stats" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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

### Sales-Specific Variables
| Variable | Description |
|----------|-------------|
| `{{name}}` | Recipient name |
| `{{email}}` | Recipient email |
| `{{order_id}}` | Order unique ID |
| `{{order_total}}` | Order total amount |
| `{{amount}}` | Payment amount |
| `{{currency}}` | Currency code |
| `{{status}}` | Order/payment status |
| `{{tracking_number}}` | Shipping tracking number |
| `{{carrier}}` | Shipping carrier |
| `{{subscription_name}}` | Subscription plan name |
| `{{next_billing_date}}` | Next billing date |
| `{{company_name}}` | Company name |

### Event Names
| Event | Description |
|-------|-------------|
| `order_confirmed` | Order confirmation |
| `order_shipped` | Order shipped notification |
| `order_delivered` | Order delivered notification |
| `order_cancelled` | Order cancellation |
| `payment_confirmed` | Payment receipt |
| `payment_failed` | Payment failure notification |
| `refund_processed` | Refund confirmation |
| `subscription_created` | New subscription welcome |
| `subscription_renewed` | Subscription renewal |
| `subscription_cancelled` | Subscription cancellation |
| `subscription_trial_ending` | Trial ending reminder |

---

## Data Models

### MailTemplate
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
