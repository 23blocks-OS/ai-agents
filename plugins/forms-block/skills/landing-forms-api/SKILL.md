---
name: landing-forms-api
description: Manage 23blocks landing form submissions via REST API. Use for lead capture, contact forms, and public form submissions with CRM integration.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Landing Forms API

Complete API reference for 23blocks landing form instance management - lead capture, contact forms, and public submissions.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://forms.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/landings/{form_id}/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /landings/:form_unique_id/instances - List Landing Instances

Lists all landing form submissions for a specific form.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/landings/form-123/instances?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `search` | string | No | Search in submissions |
| `sort` | string | No | Sort field (prefix `-` for desc) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "landing-123",
      "type": "landing_instance",
      "attributes": {
        "unique_id": "landing-123",
        "user_unique_id": null,
        "first_name": "Jane",
        "last_name": "Smith",
        "email": "jane@example.com",
        "phone": "+1-555-0456",
        "company": "Acme Corp",
        "message": "I'm interested in your services...",
        "source": "website",
        "utm_source": "google",
        "utm_medium": "cpc",
        "utm_campaign": "brand",
        "payload": {
          "product_interest": "Enterprise Plan",
          "company_size": "50-100"
        },
        "ip_address": "192.168.1.1",
        "user_agent": "Mozilla/5.0...",
        "referrer": "https://google.com",
        "crm_synced": true,
        "crm_sync_status": "success",
        "crm_sync_at": "2025-01-10T10:35:00Z",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 10,
    "totalRecords": 150
  }
}
```

---

### GET /landings/:form_unique_id/instances/:unique_id - Get Landing Instance

Retrieves a specific landing form submission.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/landings/form-123/instances/landing-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "landing-123",
    "type": "landing_instance",
    "attributes": {
      "unique_id": "landing-123",
      "first_name": "Jane",
      "last_name": "Smith",
      "email": "jane@example.com",
      "phone": "+1-555-0456",
      "company": "Acme Corp",
      "message": "I'm interested in your services...",
      "source": "website",
      "payload": {...},
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

---

### POST /landings/:form_unique_id/instances - Create Landing Instance

Creates a new landing form submission.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/landings/form-123/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "landing": {
      "first_name": "Jane",
      "last_name": "Smith",
      "email": "jane@example.com",
      "phone": "+1-555-0456",
      "company": "Acme Corp",
      "message": "I am interested in learning more about your services.",
      "source": "website",
      "selected_option": "General Inquiry",
      "form_fields": "{\"first_name\":\"Jane\",\"last_name\":\"Smith\",\"email\":\"jane@example.com\",\"message\":\"I am interested in learning more about your services.\",\"source\":\"website\"}",
      "utm_source": "google",
      "utm_medium": "cpc",
      "utm_campaign": "brand"
    }
  }'
```

**IMPORTANT:** The wrapper key is `landing` (not `landing_instance`), and `form_fields` is required as a JSON string.

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |
| `email` | string | Yes | Email address |
| `phone` | string | No | Phone number |
| `company` | string | No | Company name |
| `message` | string | No | Message/comments |
| `selected_option` | string | No | Selected option (for dropdowns) |
| `form_fields` | string | **Yes** | JSON string with all form data |
| `source` | string | No | Lead source |
| `utm_source` | string | No | UTM source |
| `utm_medium` | string | No | UTM medium |
| `utm_campaign` | string | No | UTM campaign |
| `utm_content` | string | No | UTM content |
| `utm_term` | string | No | UTM term |

**Response 201:**
```json
{
  "data": {
    "id": "new-landing-id",
    "type": "landing_instance",
    "attributes": {
      "unique_id": "new-landing-id",
      "email": "jane@example.com",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /landings/:form_unique_id/instances/:unique_id - Update Landing Instance

Updates a landing form submission.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/landings/form-123/instances/landing-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "landing": {
      "phone": "+1-555-9999",
      "notes": "Follow up on Premium Plan inquiry"
    }
  }'
```

---

### DELETE /landings/:form_unique_id/instances/:unique_id - Delete Landing Instance

Soft-deletes a landing form submission.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/landings/form-123/instances/landing-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## CRM Integration

Landing forms can sync to external CRMs (Salesforce, HubSpot, etc.).

### POST /crm/sync/landing/:unique_id - Sync to CRM

Manually triggers CRM sync for a landing instance.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/crm/sync/landing/landing-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Sync initiated",
  "status": "pending",
  "crm_record_id": null
}
```

---

## Data Models

### Landing Instance
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | string | External user ID |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `email` | string | Email address |
| `phone` | string | Phone number |
| `company` | string | Company name |
| `message` | string | Message/comments |
| `source` | string | Lead source |
| `utm_source` | string | UTM source |
| `utm_medium` | string | UTM medium |
| `utm_campaign` | string | UTM campaign |
| `utm_content` | string | UTM content |
| `utm_term` | string | UTM term |
| `payload` | object | Custom fields |
| `ip_address` | string | Submitter IP |
| `user_agent` | string | Browser info |
| `referrer` | string | Referring URL |
| `crm_synced` | boolean | CRM sync status |
| `crm_sync_status` | string | sync status detail |
| `crm_sync_at` | timestamp | Last sync time |
| `crm_record_id` | string | CRM record ID |
| `created_at` | timestamp | Creation time |

---

## Error Codes

| HTTP Status | Code | Description |
|-------------|------|-------------|
| 404 | Not Found | Landing instance not found |
| 422 | Unprocessable Entity | Validation error (missing email) |
| 400 | Bad Request | Invalid parameters |
| 409 | Conflict | Duplicate submission (if only_once enabled) |
