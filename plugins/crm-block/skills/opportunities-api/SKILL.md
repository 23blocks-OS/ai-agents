---
name: crm-opportunities-api
description: Manage 23blocks CRM opportunities via REST API. Use when creating, updating, or tracking sales opportunities through pipeline stages with value and probability tracking.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# CRM Opportunities API

Complete API reference for 23blocks CRM opportunity management through the sales pipeline.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://crm.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
echo "All credentials configured"
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | CRM API base URL | `https://crm.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/opportunities/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /opportunities/ - List Opportunities

Lists all opportunities with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/opportunities/?page=1&records=20" \
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
      "id": "opp-uuid-123",
      "type": "opportunity",
      "attributes": {
        "unique_id": "opp-uuid-123",
        "name": "Enterprise Software License",
        "account_id": "account-uuid-456",
        "value": 120000,
        "stage": "proposal",
        "close_date": "2026-06-30",
        "probability": 60,
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 24
  }
}
```

---

### GET /opportunities/:unique_id - Get Opportunity

Retrieves a single opportunity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/opportunities/opp-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "opp-uuid-123",
    "type": "opportunity",
    "attributes": {
      "unique_id": "opp-uuid-123",
      "name": "Enterprise Software License",
      "account_id": "account-uuid-456",
      "value": 120000,
      "stage": "proposal",
      "close_date": "2026-06-30",
      "probability": 60,
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    },
    "relationships": {
      "account": {
        "data": { "id": "account-uuid-456", "type": "account" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Opportunity not found

---

### POST /opportunities/ - Create Opportunity

Creates a new sales opportunity.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/opportunities/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "opportunity": {
      "name": "Enterprise Software License",
      "account_id": "account-uuid-456",
      "value": 120000,
      "stage": "discovery",
      "close_date": "2026-06-30",
      "probability": 30
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Opportunity name |
| `account_id` | uuid | No | Associated account ID |
| `value` | decimal | No | Deal value |
| `stage` | string | No | Pipeline stage (discovery, qualification, proposal, negotiation, closed_won, closed_lost) |
| `close_date` | date | No | Expected close date |
| `probability` | integer | No | Win probability (0-100) |

**Response 201:**
```json
{
  "data": {
    "id": "opp-uuid-456",
    "type": "opportunity",
    "attributes": {
      "unique_id": "opp-uuid-456",
      "name": "Enterprise Software License",
      "account_id": "account-uuid-456",
      "value": 120000,
      "stage": "discovery",
      "close_date": "2026-06-30",
      "probability": 30,
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /opportunities/:unique_id/ - Update Opportunity

Updates an existing opportunity.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/opportunities/opp-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "opportunity": {
      "stage": "negotiation",
      "probability": 80,
      "value": 135000
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "opp-uuid-123",
    "type": "opportunity",
    "attributes": {
      "unique_id": "opp-uuid-123",
      "stage": "negotiation",
      "probability": 80,
      "value": 135000,
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Opportunity not found

---

### DELETE /opportunities/:unique_id/ - Delete Opportunity

Deletes an opportunity.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/opportunities/opp-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Opportunity not found

---

## Data Models

### Opportunity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Opportunity name |
| `account_id` | uuid | Associated account ID |
| `value` | decimal | Deal value |
| `stage` | enum | discovery, qualification, proposal, negotiation, closed_won, closed_lost |
| `close_date` | date | Expected close date |
| `probability` | integer | Win probability (0-100) |
| `status` | enum | active, inactive, won, lost |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Opportunity Not Found",
    "detail": "The requested opportunity could not be found."
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
// OpportunitiesService — client.crm.opportunities
list(params?: ListOpportunitiesParams): Promise<PageResult<Opportunity>>;
get(uniqueId: string): Promise<Opportunity>;
create(data: CreateOpportunityRequest): Promise<Opportunity>;
update(uniqueId: string, data: UpdateOpportunityRequest): Promise<Opportunity>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<Opportunity>;
search(query: string, params?: ListOpportunitiesParams): Promise<PageResult<Opportunity>>;
listDeleted(params?: ListOpportunitiesParams): Promise<PageResult<Opportunity>>;
```

### TypeScript Types

```typescript
import type {
  Opportunity,
  CreateOpportunityRequest,
  UpdateOpportunityRequest,
  ListOpportunitiesParams,
} from '@23blocks/block-crm';
```

### React Hook

```typescript
import { useCrmBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCrmBlock();

  // Example: List all opportunities
  const result = await client.crm.opportunities.list();
}
```
