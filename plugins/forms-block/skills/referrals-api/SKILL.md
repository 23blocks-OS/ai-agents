---
name: referrals-api
description: Manage 23blocks referral tracking via REST API. Use for referral programs, source attribution, and reward tracking.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Referrals API

Complete API reference for 23blocks referral tracking and management.

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
curl -X GET "$BLOCKS_API_URL/referrals/{form_id}/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /referrals/:form_unique_id/instances - List Referrals

Lists all referrals for a specific form.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/referrals/form-123/instances?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
curl -X GET "$BLOCKS_API_URL/referrals/form-123/instances/ref-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

---

### POST /referrals/:form_unique_id/instances - Create Referral

Creates a new referral.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/referrals/form-123/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
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
curl -X PUT "$BLOCKS_API_URL/referrals/form-123/instances/ref-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
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
curl -X DELETE "$BLOCKS_API_URL/referrals/form-123/instances/ref-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
