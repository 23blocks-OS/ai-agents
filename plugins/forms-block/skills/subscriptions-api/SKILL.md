---
name: subscriptions-api
description: Manage 23blocks newsletter subscriptions via REST API. Use for email list management, subscribe/unsubscribe flows.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Subscriptions API

Complete API reference for 23blocks newsletter subscription management.

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

### GET /subscriptions/:form_unique_id/instances - List Subscriptions

Lists all subscriptions for a specific form.

**Request:**
```bash
curl -X GET "$API_URL/subscriptions/form-123/instances?page=1&records=20" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "sub-123",
      "type": "subscription",
      "attributes": {
        "unique_id": "sub-123",
        "email": "subscriber@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "status": "active",
        "source": "website",
        "subscribed_at": "2025-01-10T10:30:00Z",
        "unsubscribed_at": null,
        "preferences": {
          "newsletter": true,
          "product_updates": true,
          "promotions": false
        },
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

### GET /subscriptions/:form_unique_id/instances/:unique_id - Get Subscription

Retrieves a specific subscription.

**Request:**
```bash
curl -X GET "$API_URL/subscriptions/form-123/instances/sub-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

---

### POST /subscriptions/:form_unique_id/instances - Create Subscription

Creates a new subscription.

**Request:**
```bash
curl -X POST "$API_URL/subscriptions/form-123/instances" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "email": "newsubscriber@example.com",
      "first_name": "Jane",
      "last_name": "Smith",
      "source": "footer_form",
      "preferences": {
        "newsletter": true,
        "product_updates": true,
        "promotions": false
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Subscriber email |
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |
| `source` | string | No | Subscription source |
| `preferences` | object | No | Email preferences |

---

### PUT /subscriptions/:form_unique_id/instances/:unique_id - Update Subscription

Updates a subscription.

**Request:**
```bash
curl -X PUT "$API_URL/subscriptions/form-123/instances/sub-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "status": "unsubscribed",
      "preferences": {
        "newsletter": false
      }
    }
  }'
```

---

### DELETE /subscriptions/:form_unique_id/instances/:unique_id - Delete Subscription

Soft-deletes a subscription.

**Request:**
```bash
curl -X DELETE "$API_URL/subscriptions/form-123/instances/sub-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 204:** No content

---

## Data Models

### Subscription
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | Subscriber email |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `status` | enum | active, unsubscribed |
| `source` | string | Subscription source |
| `preferences` | object | Email preferences |
| `subscribed_at` | timestamp | Subscription time |
| `unsubscribed_at` | timestamp | Unsubscribe time |
| `created_at` | timestamp | Creation time |
