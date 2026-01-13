---
name: referrals-api
description: Manage 23blocks referral tracking via REST API. Use for referral programs, source attribution, and reward tracking.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Referrals API

Complete API reference for 23blocks referral tracking and management.

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

### GET /referrals/:form_unique_id/instances - List Referrals

Lists all referrals for a specific form.

**Request:**
```bash
curl -X GET "$API_URL/referrals/form-123/instances?page=1&records=20" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "ref-123",
      "type": "referral",
      "attributes": {
        "unique_id": "ref-123",
        "referrer_unique_id": "user-456",
        "referrer_email": "referrer@example.com",
        "referrer_name": "John Referrer",
        "referred_email": "referred@example.com",
        "referred_name": "Jane Referred",
        "status": "pending",
        "referral_code": "JOHN2025",
        "source": "email_campaign",
        "reward_status": "pending",
        "reward_amount": null,
        "metadata": {
          "campaign": "winter_promo",
          "channel": "email"
        },
        "converted_at": null,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### GET /referrals/:form_unique_id/instances/:unique_id - Get Referral

Retrieves a specific referral.

**Request:**
```bash
curl -X GET "$API_URL/referrals/form-123/instances/ref-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

---

### POST /referrals/:form_unique_id/instances - Create Referral

Creates a new referral.

**Request:**
```bash
curl -X POST "$API_URL/referrals/form-123/instances" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "referral": {
      "referrer_unique_id": "user-456",
      "referrer_email": "referrer@example.com",
      "referrer_name": "John Referrer",
      "referred_email": "newuser@example.com",
      "referred_name": "New User",
      "referral_code": "JOHN2025",
      "source": "share_link",
      "metadata": {
        "campaign": "spring_launch",
        "product": "premium"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `referrer_unique_id` | string | No | Referrer user ID |
| `referrer_email` | string | Yes | Referrer email |
| `referrer_name` | string | No | Referrer name |
| `referred_email` | string | Yes | Referred person email |
| `referred_name` | string | No | Referred person name |
| `referral_code` | string | No | Referral code used |
| `source` | string | No | Referral source |
| `metadata` | object | No | Custom metadata |

---

### PUT /referrals/:form_unique_id/instances/:unique_id - Update Referral

Updates a referral (e.g., mark as converted).

**Request:**
```bash
curl -X PUT "$API_URL/referrals/form-123/instances/ref-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "referral": {
      "status": "converted",
      "reward_status": "pending",
      "reward_amount": 25.00
    }
  }'
```

---

### DELETE /referrals/:form_unique_id/instances/:unique_id - Delete Referral

Soft-deletes a referral.

**Request:**
```bash
curl -X DELETE "$API_URL/referrals/form-123/instances/ref-123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "AppId: $APP_ID"
```

**Response 204:** No content

---

## Data Models

### Referral
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `referrer_unique_id` | string | Referrer user ID |
| `referrer_email` | string | Referrer email |
| `referrer_name` | string | Referrer name |
| `referred_email` | string | Referred email |
| `referred_name` | string | Referred name |
| `status` | enum | pending, converted, expired |
| `referral_code` | string | Referral code |
| `source` | string | Referral source |
| `reward_status` | enum | pending, paid, cancelled |
| `reward_amount` | decimal | Reward amount |
| `metadata` | object | Custom metadata |
| `converted_at` | timestamp | Conversion time |
| `created_at` | timestamp | Creation time |
