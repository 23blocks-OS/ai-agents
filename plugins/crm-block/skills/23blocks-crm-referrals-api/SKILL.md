---
name: 23blocks-crm-referrals-api
description: Manage 23blocks CRM referrals via REST API. Use when creating, tracking, or managing referral programs with reward status tracking.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Referrals API

Complete API reference for 23blocks CRM referral management with reward tracking.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Crm API base URL | `https://crm.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://crm.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://crm.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### GET /referrals/ - List Referrals

Lists all referrals with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/referrals/?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "referral-uuid-123",
      "type": "referral",
      "attributes": {
        "unique_id": "referral-uuid-123",
        "referrer_id": "contact-uuid-456",
        "referred_name": "Charlie Brown",
        "referred_email": "charlie@example.com",
        "source": "client_referral",
        "reward_status": "pending",
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 22
  }
}
```

---

### GET /referrals/:unique_id - Get Referral

Retrieves a single referral by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/referrals/referral-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "referral-uuid-123",
    "type": "referral",
    "attributes": {
      "unique_id": "referral-uuid-123",
      "referrer_id": "contact-uuid-456",
      "referred_name": "Charlie Brown",
      "referred_email": "charlie@example.com",
      "source": "client_referral",
      "reward_status": "pending",
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    },
    "relationships": {
      "referrer": {
        "data": { "id": "contact-uuid-456", "type": "contact" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Referral not found

---

### POST /referrals/ - Create Referral

Creates a new referral.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/referrals/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "referral": {
      "referrer_id": "contact-uuid-456",
      "referred_name": "Diana Prince",
      "referred_email": "diana@example.com",
      "source": "client_referral"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `referrer_id` | uuid | Yes | ID of the referring contact |
| `referred_name` | string | Yes | Name of the referred person |
| `referred_email` | string | Yes | Email of the referred person |
| `source` | string | No | Referral source |

**Response 201:**
```json
{
  "data": {
    "id": "referral-uuid-456",
    "type": "referral",
    "attributes": {
      "unique_id": "referral-uuid-456",
      "referrer_id": "contact-uuid-456",
      "referred_name": "Diana Prince",
      "referred_email": "diana@example.com",
      "source": "client_referral",
      "reward_status": "pending",
      "status": "active",
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /referrals/:unique_id/ - Update Referral

Updates an existing referral.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/referrals/referral-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "referral": {
      "reward_status": "awarded",
      "status": "converted"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "referral-uuid-123",
    "type": "referral",
    "attributes": {
      "unique_id": "referral-uuid-123",
      "reward_status": "awarded",
      "status": "converted",
      "updated_at": "2026-02-15T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Referral not found

---

### DELETE /referrals/:unique_id/ - Delete Referral

Deletes a referral.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/referrals/referral-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Referral not found

---

## Data Models

### Referral
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `referrer_id` | uuid | ID of the referring contact |
| `referred_name` | string | Name of the referred person |
| `referred_email` | string | Email of the referred person |
| `source` | string | Referral source |
| `reward_status` | enum | pending, awarded, expired |
| `status` | enum | active, converted, lost, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Referral Not Found",
    "detail": "The requested referral could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-crm
```

### Setup

```typescript
import { create23BlocksClient } from '@23blocks/sdk';

const client = create23BlocksClient({
  authToken: process.env.BLOCKS_AUTH_TOKEN!,
  apiKey: process.env.BLOCKS_API_KEY!,
  apiUrl: process.env.BLOCKS_API_URL!,
});
```

### Available Methods

```typescript
// ReferralsService — client.crm.referrals
list(params?: ListReferralsParams): Promise<PageResult<Referral>>;
get(uniqueId: string): Promise<Referral>;
create(data: CreateReferralRequest): Promise<Referral>;
update(uniqueId: string, data: UpdateReferralRequest): Promise<Referral>;
delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Referral,
  CreateReferralRequest,
  UpdateReferralRequest,
  ListReferralsParams,
} from '@23blocks/block-crm';
```

### React Hook

```typescript
import { useCrmBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCrmBlock();

  // Example: List all referrals
  const result = await client.crm.referrals.list();
}
```
