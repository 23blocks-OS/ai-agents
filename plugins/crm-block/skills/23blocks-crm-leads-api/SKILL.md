---
name: 23blocks-crm-leads-api
description: Manage 23blocks CRM leads via REST API. Use when creating, updating, or tracking leads through the sales pipeline, and managing lead follow-up tasks.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Leads API

Complete API reference for 23blocks CRM lead management with follow-up task tracking.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | CRM API base URL | `https://crm.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/leads/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/leads/` | List leads (paginated) |
| GET | `/leads/:unique_id` | Get lead by ID |
| POST | `/leads/` | Create lead |
| PUT | `/leads/:unique_id/` | Update lead |
| DELETE | `/leads/:unique_id/` | Delete lead |
| GET | `/leads/:unique_id/follows` | List follow-up tasks |
| GET | `/leads/:unique_id/follows/:follow_unique_id` | Get follow-up task |
| POST | `/leads/:unique_id/follows` | Create follow-up task |
| PUT | `/leads/:unique_id/follows/:follow_unique_id` | Update follow-up task |
| DELETE | `/leads/:unique_id/follows/:follow_unique_id` | Delete follow-up task |

---

## Data Models

### Lead
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Lead name |
| `source` | string | Lead source (website, referral, cold_call, etc.) |
| `account_id` | uuid | Associated account ID |
| `contact_id` | uuid | Associated contact ID |
| `value` | decimal | Estimated deal value |
| `stage` | string | Pipeline stage |
| `probability` | integer | Win probability (0-100) |
| `status` | enum | active, inactive, converted, lost |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Follow
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Task title |
| `description` | string | Task description |
| `due_date` | timestamp | Due date |
| `status` | enum | pending, completed, overdue |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Lead Not Found",
    "detail": "The requested lead could not be found."
  }]
}
```

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
// LeadsService — client.crm.leads
list(params?: ListLeadsParams): Promise<PageResult<Lead>>;
get(uniqueId: string): Promise<Lead>;
create(data: CreateLeadRequest): Promise<Lead>;
update(uniqueId: string, data: UpdateLeadRequest): Promise<Lead>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<Lead>;
search(query: string, params?: ListLeadsParams): Promise<PageResult<Lead>>;
listDeleted(params?: ListLeadsParams): Promise<PageResult<Lead>>;
```

### TypeScript Types

```typescript
import type {
  Lead,
  CreateLeadRequest,
  UpdateLeadRequest,
  ListLeadsParams,
} from '@23blocks/block-crm';
```

### React Hook

```typescript
import { useCrmBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCrmBlock();

  // Example: List all leads
  const result = await client.crm.leads.list();
}
```
