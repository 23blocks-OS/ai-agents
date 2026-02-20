---
name: assets-audits-api
description: Create and manage 23blocks asset audits via REST API. Use when creating, listing, updating, or deleting audit entries on assets, including webhook-based audit creation for external systems.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Audits API

Complete API reference for 23blocks Assets Block audit management with webhook support.

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
curl -X GET "$BLOCKS_API_URL/assets/asset-uuid-123/audits" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /assets/:unique_id/audits - List Audits

Lists all audits for an asset.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assets/asset-uuid-123/audits" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "audit-uuid-001",
      "type": "asset_audit",
      "attributes": {
        "unique_id": "audit-uuid-001",
        "asset_unique_id": "asset-uuid-123",
        "audit_type": "inventory_check",
        "status": "passed",
        "notes": "All components accounted for and in good condition",
        "audited_by_unique_id": "user-uuid-456",
        "audited_by_name": "John Doe",
        "audited_at": "2025-03-01T09:00:00Z",
        "payload": {},
        "created_at": "2025-03-01T09:30:00Z",
        "updated_at": "2025-03-01T09:30:00Z"
      }
    },
    {
      "id": "audit-uuid-002",
      "type": "asset_audit",
      "attributes": {
        "unique_id": "audit-uuid-002",
        "asset_unique_id": "asset-uuid-123",
        "audit_type": "compliance",
        "status": "failed",
        "notes": "Missing compliance sticker",
        "audited_by_unique_id": "user-uuid-789",
        "audited_by_name": "Jane Smith",
        "audited_at": "2025-02-15T14:00:00Z",
        "payload": {},
        "created_at": "2025-02-15T14:30:00Z",
        "updated_at": "2025-02-15T14:30:00Z"
      }
    }
  ]
}
```

---

### GET /assets/:unique_id/audits/:audit_unique_id - Get Audit

Retrieves a single audit by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assets/asset-uuid-123/audits/audit-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "audit-uuid-001",
    "type": "asset_audit",
    "attributes": {
      "unique_id": "audit-uuid-001",
      "asset_unique_id": "asset-uuid-123",
      "audit_type": "inventory_check",
      "status": "passed",
      "notes": "All components accounted for and in good condition",
      "audited_by_unique_id": "user-uuid-456",
      "audited_by_name": "John Doe",
      "audited_at": "2025-03-01T09:00:00Z",
      "payload": {
        "checklist_items": 12,
        "checklist_passed": 12
      },
      "created_at": "2025-03-01T09:30:00Z",
      "updated_at": "2025-03-01T09:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Audit not found

---

### POST /assets/:unique_id/audits - Create Audit

Creates a new audit entry on an asset.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assets/asset-uuid-123/audits" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "audit": {
      "audit_type": "inventory_check",
      "status": "passed",
      "notes": "All components verified and accounted for",
      "audited_at": "2025-03-01T09:00:00Z",
      "payload": {
        "checklist_items": 12,
        "checklist_passed": 12
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `audit_type` | string | Yes | Type of audit (e.g., inventory_check, compliance, safety, condition) |
| `status` | enum | Yes | passed, failed, pending, in_progress |
| `notes` | string | No | Audit notes and observations |
| `audited_at` | timestamp | No | When audit was performed (defaults to now) |
| `payload` | json | No | Custom audit data |

**Response 201:**
```json
{
  "data": {
    "id": "new-audit-uuid",
    "type": "asset_audit",
    "attributes": {
      "unique_id": "new-audit-uuid",
      "asset_unique_id": "asset-uuid-123",
      "audit_type": "inventory_check",
      "status": "passed",
      "notes": "All components verified and accounted for",
      "audited_by_unique_id": "current-user-uuid",
      "audited_at": "2025-03-01T09:00:00Z",
      "created_at": "2025-03-01T09:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Asset not found
- `422 Unprocessable Entity` - Validation errors

---

### POST /assets/:unique_id/audits/:audit_unique_id - Update Audit Entry

Updates an existing audit entry.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assets/asset-uuid-123/audits/audit-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "audit": {
      "status": "passed",
      "notes": "Re-audit completed successfully after remediation",
      "payload": {
        "checklist_items": 12,
        "checklist_passed": 12,
        "remediation_completed": true
      }
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "audit-uuid-001",
    "type": "asset_audit",
    "attributes": {
      "unique_id": "audit-uuid-001",
      "status": "passed",
      "notes": "Re-audit completed successfully after remediation",
      "updated_at": "2025-03-05T10:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Audit not found

---

### DELETE /assets/:unique_id/audits/:audit_unique_id - Delete Audit

Deletes an audit entry.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/assets/asset-uuid-123/audits/audit-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Audit not found

---

## Webhook

### POST /companies/:url_id/assets/:unique_id/audits - Webhook Audit

Creates an audit entry via company webhook (for external systems).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/my-company/assets/asset-uuid-123/audits" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "audit": {
      "audit_type": "automated_scan",
      "status": "passed",
      "notes": "Automated compliance scan completed",
      "audited_at": "2025-03-01T09:00:00Z",
      "payload": {
        "scan_tool": "AssetScanner v3.2",
        "vulnerabilities_found": 0
      }
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "webhook-audit-uuid",
    "type": "asset_audit",
    "attributes": {
      "unique_id": "webhook-audit-uuid",
      "asset_unique_id": "asset-uuid-123",
      "audit_type": "automated_scan",
      "status": "passed",
      "source": "webhook",
      "created_at": "2025-03-01T09:30:00Z"
    }
  }
}
```

---

## Data Models

### AssetAudit
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `asset_unique_id` | uuid | Parent asset ID |
| `audit_type` | string | Type of audit (inventory_check, compliance, safety, condition, automated_scan) |
| `status` | enum | passed, failed, pending, in_progress |
| `notes` | string | Audit notes and observations |
| `audited_by_unique_id` | uuid | User who performed the audit |
| `audited_by_name` | string | Name of the auditor |
| `audited_at` | timestamp | When audit was performed |
| `source` | string | Source of audit (manual, webhook) |
| `payload` | jsonb | Custom audit data |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Audit Not Found",
    "detail": "The requested audit could not be found."
  }]
}
```

| Code | Description |
|------|-------------|
| `401` | Unauthorized - Invalid or missing credentials |
| `404` | Not Found - Asset or audit not found |
| `422` | Unprocessable Entity - Validation errors |

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-assets
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
// AssetAuditsService — client.assets.assetAudits
client.assets.assetAudits.list(assetUniqueId: string, params?: ListAssetAuditsParams): Promise<PageResult<AssetAudit>>;
client.assets.assetAudits.get(assetUniqueId: string, auditUniqueId: string): Promise<AssetAudit>;
client.assets.assetAudits.create(assetUniqueId: string, data: CreateAssetAuditRequest): Promise<AssetAudit>;
client.assets.assetAudits.update(assetUniqueId: string, auditUniqueId: string, data: UpdateAssetAuditRequest): Promise<AssetAudit>;
client.assets.assetAudits.delete(assetUniqueId: string, auditUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  AssetAudit,
  CreateAssetAuditRequest,
  UpdateAssetAuditRequest,
  ListAssetAuditsParams,
} from '@23blocks/block-assets';
```

### React Hook

```typescript
import { useAssetsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAssetsBlock();
  const result = await client.assets.assetAudits.list('asset-unique-id');
}
```
