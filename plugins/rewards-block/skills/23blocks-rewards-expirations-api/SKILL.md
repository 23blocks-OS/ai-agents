---
name: 23blocks-rewards-expirations-api
description: "Rewards Block point expiration policies: duration, grace period, notices. Use when deciding how long points last."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Expirations API

Complete API reference for 23blocks point expiration rule management with configurable durations, grace periods, and notification settings.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://rewards.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"rule_in_use","title":"Expiration Rule In Use","detail":"This expiration rule is currently attached to earning rules and cannot be deleted."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-rewards`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useRewardsBlock()` from `@23blocks/react`.

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
