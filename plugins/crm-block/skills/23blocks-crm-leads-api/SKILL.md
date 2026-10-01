---
name: 23blocks-crm-leads-api
description: "CRM Block leads and their follow-up tasks. Use when capturing or working a lead through the pipeline."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Leads API

Complete API reference for 23blocks CRM lead management with follow-up task tracking.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://crm.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Lead Not Found","detail":"The requested lead could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-crm`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCrmBlock()` from `@23blocks/react`.

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
