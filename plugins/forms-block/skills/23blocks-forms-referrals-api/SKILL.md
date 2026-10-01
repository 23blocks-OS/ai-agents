---
name: 23blocks-forms-referrals-api
description: "Forms Block referral instances: attribution and rewards. Use for form-based referral programs."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Referrals API

Complete API reference for 23blocks referral tracking and management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://forms.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /referrals/:form_unique_id/instances - List Referrals

Lists all referrals for a specific form.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/referrals/form-123/instances?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

---

### POST /referrals/:form_unique_id/instances - Create Referral

Creates a new referral.

> **Public Endpoint:** This endpoint does NOT require authentication. The `Authorization` and `X-API-KEY` headers are optional. Public submissions are accepted without auth.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/referrals/form-123/instances" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Failed","detail":"Referrer email can't be blank."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-forms`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// ReferralsService — client.forms.referrals
list(formUniqueId: string, params?: ListReferralsParams): Promise<PageResult<Referral>>;
get(formUniqueId: string, uniqueId: string): Promise<Referral>;
create(formUniqueId: string, data: CreateReferralRequest): Promise<Referral>;
update(formUniqueId: string, uniqueId: string, data: UpdateReferralRequest): Promise<Referral>;
delete(formUniqueId: string, uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Referral,
  CreateReferralRequest,
  UpdateReferralRequest,
  ListReferralsParams,
} from '@23blocks/block-forms';
```

### React Hook

```typescript
import { useFormsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFormsBlock();
  const result = await client.forms.referrals.list('form-unique-id');
}
```
