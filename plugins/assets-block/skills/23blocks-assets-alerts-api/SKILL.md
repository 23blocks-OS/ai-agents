---
name: 23blocks-assets-alerts-api
description: "Assets Block alerts: get, evaluate and delete monitoring alerts on assets. Use when checking, evaluating or clearing asset alerts."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Alerts API

Complete API reference for 23blocks Assets Block alert management and evaluation.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://assets.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /alerts/:unique_id - Get Alert

Retrieves a single alert by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/alerts/alert-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "alert-uuid-123",
    "type": "alert",
    "attributes": {
      "unique_id": "alert-uuid-123",
      "alert_type": "maintenance_due",
      "severity": "warning",
      "title": "Maintenance Due Soon",
      "description": "Asset 'Laptop Dell XPS 15' has maintenance due in 7 days",
      "asset_unique_id": "asset-uuid-456",
      "asset_name": "Laptop Dell XPS 15",
      "status": "active",
      "triggered_at": "2025-03-08T00:00:00Z",
      "acknowledged_at": null,
      "resolved_at": null,
      "payload": {
        "maintenance_due_at": "2025-03-15T09:00:00Z",
        "days_remaining": 7
      },
      "created_at": "2025-03-08T00:00:00Z",
      "updated_at": "2025-03-08T00:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Alert not found

---

### POST /alerts/eval - Evaluate Alerts

Evaluates alert conditions across assets and triggers applicable alerts.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/alerts/eval" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "evaluation": {
      "alert_types": ["maintenance_due", "warranty_expiry", "overdue_return"],
      "scope": "all"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `alert_types` | array | No | Types to evaluate (defaults to all). Options: maintenance_due, warranty_expiry, overdue_return, condition_degraded, audit_failed |
| `scope` | string | No | Evaluation scope: all, active_only (default: all) |

**Response 200:**
```json
{
  "data": {
    "evaluated_count": 150,
    "alerts_triggered": 5,
    "alerts": [
      {
        "id": "new-alert-uuid-1",
        "type": "alert",
        "attributes": {
          "unique_id": "new-alert-uuid-1",
          "alert_type": "maintenance_due",
          "severity": "warning",
          "title": "Maintenance Due Soon",
          "asset_unique_id": "asset-uuid-001",
          "asset_name": "Laptop Dell XPS 15",
          "status": "active",
          "triggered_at": "2025-03-15T10:00:00Z"
        }
      },
      {
        "id": "new-alert-uuid-2",
        "type": "alert",
        "attributes": {
          "unique_id": "new-alert-uuid-2",
          "alert_type": "warranty_expiry",
          "severity": "info",
          "title": "Warranty Expiring Soon",
          "asset_unique_id": "asset-uuid-002",
          "asset_name": "Monitor Samsung 27",
          "status": "active",
          "triggered_at": "2025-03-15T10:00:00Z"
        }
      }
    ]
  }
}
```

---

### DELETE /alerts/:unique_id - Delete Alert

Deletes an alert.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/alerts/alert-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** Returns `{}` with status 204

**Errors:**
- `404 Not Found` - Alert not found

---

## Data Models

### Alert
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `alert_type` | string | Type of alert (maintenance_due, warranty_expiry, overdue_return, condition_degraded, audit_failed) |
| `severity` | enum | info, warning, critical |
| `title` | string | Alert title |
| `description` | string | Alert description |
| `asset_unique_id` | uuid | Associated asset ID |
| `asset_name` | string | Associated asset name |
| `status` | enum | active, acknowledged, resolved |
| `triggered_at` | timestamp | When the alert was triggered |
| `acknowledged_at` | timestamp | When the alert was acknowledged |
| `resolved_at` | timestamp | When the alert was resolved |
| `payload` | jsonb | Custom alert data |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Alert Types
| Type | Description |
|------|-------------|
| `maintenance_due` | Asset maintenance is due or overdue |
| `warranty_expiry` | Asset warranty is expiring soon |
| `overdue_return` | Lent asset has not been returned by due date |
| `condition_degraded` | Asset condition has been marked as poor |
| `audit_failed` | Asset failed its most recent audit |

### Severity Levels
| Level | Description |
|-------|-------------|
| `info` | Informational, no immediate action required |
| `warning` | Action should be taken soon |
| `critical` | Immediate action required |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Alert Not Found",
    "detail": "The requested alert could not be found."
  }]
}
```

| Code | Description |
|------|-------------|
| `401` | Unauthorized - Invalid or missing credentials |
| `404` | Not Found - Alert not found |

---

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-assets`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// AlertsService — client.assets.alerts
client.assets.alerts.get(uniqueId: string): Promise<AssetAlert>;
client.assets.alerts.create(data: CreateAssetAlertRequest): Promise<AssetAlert>;
client.assets.alerts.delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  AssetAlert,
  CreateAssetAlertRequest,
} from '@23blocks/block-assets';
```

### React Hook

```typescript
import { useAssetsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAssetsBlock();
  const result = await client.assets.alerts.get('alert-unique-id');
}
```
