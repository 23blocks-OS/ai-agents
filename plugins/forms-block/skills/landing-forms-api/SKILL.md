---
name: landing-forms-api
description: Manage 23blocks landing form submissions via REST API. Use for lead capture, contact forms, and public form submissions with CRM integration.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Landing Forms API

Complete API reference for 23blocks landing form instance management - lead capture, contact forms, and public submissions.

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

### GET /landings/:form_unique_id/instances - List Landing Instances

Lists all landing form submissions for a specific form.

**Request:**
```bash
curl -X GET "$API_URL/landings/form-123/instances?page=1&records=20" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
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
curl -X GET "$API_URL/landings/form-123/instances/landing-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
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
curl -X POST "$API_URL/landings/form-123/instances" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "landing_instance": {
      "first_name": "Jane",
      "last_name": "Smith",
      "email": "jane@example.com",
      "phone": "+1-555-0456",
      "company": "Acme Corp",
      "message": "I am interested in learning more about your services.",
      "source": "website",
      "utm_source": "google",
      "utm_medium": "cpc",
      "utm_campaign": "brand",
      "payload": {
        "product_interest": "Enterprise Plan",
        "company_size": "50-100",
        "budget": "$10k-$50k"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |
| `email` | string | Yes | Email address |
| `phone` | string | No | Phone number |
| `company` | string | No | Company name |
| `message` | string | No | Message/comments |
| `source` | string | No | Lead source |
| `utm_source` | string | No | UTM source |
| `utm_medium` | string | No | UTM medium |
| `utm_campaign` | string | No | UTM campaign |
| `utm_content` | string | No | UTM content |
| `utm_term` | string | No | UTM term |
| `payload` | object | No | Custom fields |

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
curl -X PUT "$API_URL/landings/form-123/instances/landing-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "landing_instance": {
      "phone": "+1-555-9999",
      "payload": {
        "product_interest": "Premium Plan",
        "follow_up_date": "2025-01-20"
      }
    }
  }'
```

---

### DELETE /landings/:form_unique_id/instances/:unique_id - Delete Landing Instance

Soft-deletes a landing form submission.

**Request:**
```bash
curl -X DELETE "$API_URL/landings/form-123/instances/landing-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 204:** No content

---

## CRM Integration

Landing forms can sync to external CRMs (Salesforce, HubSpot, etc.).

### POST /crm/sync/landing/:unique_id - Sync to CRM

Manually triggers CRM sync for a landing instance.

**Request:**
```bash
curl -X POST "$API_URL/crm/sync/landing/landing-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
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
