---
name: assets-operations-api
description: Create and manage 23blocks asset operations via REST API. Use when recording, listing, or retrieving operational records on assets, including webhook-based operation creation for external systems.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Operations API

Complete API reference for 23blocks Assets Block operation management with webhook support.

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
curl -X GET "$BLOCKS_API_URL/assets/asset-uuid-123/operations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /assets/:unique_id/operations - List Operations

Lists all operations for an asset.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assets/asset-uuid-123/operations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "operation-uuid-001",
      "type": "asset_operation",
      "attributes": {
        "unique_id": "operation-uuid-001",
        "asset_unique_id": "asset-uuid-123",
        "operation_type": "checkout",
        "description": "Asset checked out for remote work",
        "performed_by_unique_id": "user-uuid-456",
        "performed_by_name": "John Doe",
        "performed_at": "2025-03-01T09:00:00Z",
        "payload": {},
        "created_at": "2025-03-01T09:30:00Z",
        "updated_at": "2025-03-01T09:30:00Z"
      }
    },
    {
      "id": "operation-uuid-002",
      "type": "asset_operation",
      "attributes": {
        "unique_id": "operation-uuid-002",
        "asset_unique_id": "asset-uuid-123",
        "operation_type": "checkin",
        "description": "Asset returned to office",
        "performed_by_unique_id": "user-uuid-456",
        "performed_by_name": "John Doe",
        "performed_at": "2025-03-10T08:00:00Z",
        "payload": {},
        "created_at": "2025-03-10T08:30:00Z",
        "updated_at": "2025-03-10T08:30:00Z"
      }
    }
  ]
}
```

---

### GET /assets/:unique_id/operations/:operation_unique_id - Get Operation

Retrieves a single operation by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assets/asset-uuid-123/operations/operation-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "operation-uuid-001",
    "type": "asset_operation",
    "attributes": {
      "unique_id": "operation-uuid-001",
      "asset_unique_id": "asset-uuid-123",
      "operation_type": "checkout",
      "description": "Asset checked out for remote work",
      "performed_by_unique_id": "user-uuid-456",
      "performed_by_name": "John Doe",
      "performed_at": "2025-03-01T09:00:00Z",
      "payload": {
        "location": "Home Office",
        "expected_return": "2025-03-10"
      },
      "created_at": "2025-03-01T09:30:00Z",
      "updated_at": "2025-03-01T09:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Operation not found

---

### POST /assets/:unique_id/operations - Create Operation

Creates a new operation record on an asset.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assets/asset-uuid-123/operations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "operation": {
      "operation_type": "checkout",
      "description": "Asset checked out for client demo",
      "performed_at": "2025-03-15T10:00:00Z",
      "payload": {
        "purpose": "Client presentation",
        "expected_return": "2025-03-16"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation_type` | string | Yes | Type of operation (e.g., checkout, checkin, transfer, deployment, decommission) |
| `description` | string | No | Operation description |
| `performed_at` | timestamp | No | When the operation was performed (defaults to now) |
| `payload` | json | No | Custom operation data |

**Response 201:**
```json
{
  "data": {
    "id": "new-operation-uuid",
    "type": "asset_operation",
    "attributes": {
      "unique_id": "new-operation-uuid",
      "asset_unique_id": "asset-uuid-123",
      "operation_type": "checkout",
      "description": "Asset checked out for client demo",
      "performed_by_unique_id": "current-user-uuid",
      "performed_at": "2025-03-15T10:00:00Z",
      "created_at": "2025-03-15T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Asset not found
- `422 Unprocessable Entity` - Validation errors

---

## Webhook

### POST /companies/:url_id/assets/:unique_id/operations - Webhook Operation

Creates an operation record via company webhook (for external systems).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/my-company/assets/asset-uuid-123/operations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "operation": {
      "operation_type": "automated_scan",
      "description": "Automated network scan detected asset",
      "performed_at": "2025-03-15T10:00:00Z",
      "payload": {
        "scan_tool": "NetworkScanner v2.1",
        "ip_address": "192.168.1.45",
        "mac_address": "AA:BB:CC:DD:EE:FF"
      }
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "webhook-operation-uuid",
    "type": "asset_operation",
    "attributes": {
      "unique_id": "webhook-operation-uuid",
      "asset_unique_id": "asset-uuid-123",
      "operation_type": "automated_scan",
      "source": "webhook",
      "created_at": "2025-03-15T10:30:00Z"
    }
  }
}
```

---

## Data Models

### AssetOperation
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `asset_unique_id` | uuid | Parent asset ID |
| `operation_type` | string | Type of operation (checkout, checkin, transfer, deployment, decommission, automated_scan) |
| `description` | string | Operation description |
| `performed_by_unique_id` | uuid | User who performed the operation |
| `performed_by_name` | string | Name of the performer |
| `performed_at` | timestamp | When the operation was performed |
| `source` | string | Source of operation (manual, webhook) |
| `payload` | jsonb | Custom operation data |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Operation Not Found",
    "detail": "The requested operation could not be found."
  }]
}
```

| Code | Description |
|------|-------------|
| `401` | Unauthorized - Invalid or missing credentials |
| `404` | Not Found - Asset or operation not found |
| `422` | Unprocessable Entity - Validation errors |
