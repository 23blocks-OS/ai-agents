---
name: 23blocks-rewards-expirations-api
description: Manage 23blocks point expiration rules via REST API. Use when creating, updating, or deleting expiration policies that control how long earned points remain valid, including grace periods and advance notifications.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Expirations API

Complete API reference for 23blocks point expiration rule management with configurable durations, grace periods, and notification settings.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Rewards API base URL | `https://rewards.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://rewards.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://rewards.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### GET /expirations - List Expiration Rules

Lists all expiration rules with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/expirations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
      "id": "expiration-uuid-123",
      "type": "expiration_rule",
      "attributes": {
        "unique_id": "expiration-uuid-123",
        "name": "Standard 365-Day Expiry",
        "expiration_type": "rolling",
        "duration_days": 365,
        "grace_period_days": 30,
        "notification_days_before": 14,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 5
  }
}
```

---

### GET /expirations/:unique_id - Get Expiration Rule

Retrieves a single expiration rule by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/expirations/expiration-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "expiration-uuid-123",
    "type": "expiration_rule",
    "attributes": {
      "unique_id": "expiration-uuid-123",
      "name": "Standard 365-Day Expiry",
      "expiration_type": "rolling",
      "duration_days": 365,
      "grace_period_days": 30,
      "notification_days_before": 14,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Expiration rule not found

---

### POST /expirations/ - Create Expiration Rule

Creates a new point expiration rule.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/expirations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rule": {
      "name": "Standard 365-Day Expiry",
      "expiration_type": "rolling",
      "duration_days": 365,
      "grace_period_days": 30,
      "notification_days_before": 14
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Rule name |
| `expiration_type` | string | Yes | Expiration type (rolling, fixed, calendar_year) |
| `duration_days` | integer | Yes | Days until points expire |
| `grace_period_days` | integer | No | Additional grace period days (default: 0) |
| `notification_days_before` | integer | No | Days before expiry to notify customer (default: 0) |

**Response 201:**
```json
{
  "data": {
    "id": "new-expiration-uuid",
    "type": "expiration_rule",
    "attributes": {
      "unique_id": "new-expiration-uuid",
      "name": "Standard 365-Day Expiry",
      "expiration_type": "rolling",
      "duration_days": 365,
      "grace_period_days": 30,
      "notification_days_before": 14,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /expirations/:unique_id - Update Expiration Rule

Updates an existing expiration rule.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/expirations/expiration-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rule": {
      "duration_days": 180,
      "grace_period_days": 15,
      "notification_days_before": 7
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "expiration-uuid-123",
    "type": "expiration_rule",
    "attributes": {
      "unique_id": "expiration-uuid-123",
      "name": "Standard 365-Day Expiry",
      "expiration_type": "rolling",
      "duration_days": 180,
      "grace_period_days": 15,
      "notification_days_before": 7,
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Expiration rule not found

---

### DELETE /expirations/:unique_id - Delete Expiration Rule

Deletes an expiration rule.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/expirations/expiration-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Expiration rule not found
- `422 Unprocessable Entity` - Rule is currently in use by earning rules

---

## Data Models

### ExpirationRule
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Rule name |
| `expiration_type` | string | rolling (from earn date), fixed (specific date), calendar_year (end of year) |
| `duration_days` | integer | Days until points expire |
| `grace_period_days` | integer | Additional grace period after expiry date |
| `notification_days_before` | integer | Days before expiry to send notification |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "rule_in_use",
    "title": "Expiration Rule In Use",
    "detail": "This expiration rule is currently attached to earning rules and cannot be deleted."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-rewards
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
// ExpirationRulesService — client.rewards.expirationRules
client.rewards.expirationRules.list(params?: ListExpirationRulesParams): Promise<PageResult<ExpirationRule>>;
client.rewards.expirationRules.get(uniqueId: string): Promise<ExpirationRule>;
client.rewards.expirationRules.create(data: CreateExpirationRuleRequest): Promise<ExpirationRule>;
client.rewards.expirationRules.update(uniqueId: string, data: UpdateExpirationRuleRequest): Promise<ExpirationRule>;
client.rewards.expirationRules.delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  ExpirationRule,
  CreateExpirationRuleRequest,
  UpdateExpirationRuleRequest,
  ListExpirationRulesParams,
} from '@23blocks/block-rewards';
```

### React Hook

```typescript
import { useRewardsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useRewardsBlock();
  const result = await client.rewards.expirationRules.list();
}
```
