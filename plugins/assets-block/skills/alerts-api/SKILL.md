---
name: assets-alerts-api
description: Manage 23blocks asset alerts via REST API. Use when retrieving, evaluating, or deleting alerts on assets for monitoring and notification purposes.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Alerts API

Complete API reference for 23blocks Assets Block alert management and evaluation.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://assets.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Assets API base URL | `https://assets.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/alerts/alert-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /alerts/:unique_id - Get Alert

Retrieves a single alert by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/alerts/alert-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY" \
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
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

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
