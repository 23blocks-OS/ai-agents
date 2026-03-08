---
name: 23blocks-crm-calendar-sync-api
description: Manage 23blocks CRM calendar sync via REST API. Use when connecting calendar accounts, syncing calendars, managing busy blocks, generating ICS tokens, serving public ICS feeds, and handling Cal.com webhooks.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Calendar Sync API

Complete API reference for 23blocks CRM calendar synchronization with external providers, busy blocks, ICS feeds, and Cal.com integration.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | CRM API base URL | `https://crm.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/calendar_accounts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users/:user_unique_id/calendar_accounts` | List calendar accounts |
| GET | `/users/:user_unique_id/calendar_accounts/:id` | Get calendar account |
| POST | `/users/:user_unique_id/calendar_accounts` | Create calendar account |
| PUT | `/users/:user_unique_id/calendar_accounts/:id` | Update calendar account |
| DELETE | `/users/:user_unique_id/calendar_accounts/:id` | Delete calendar account |
| POST | `/users/:user_unique_id/calendar/sync` | Sync user calendar |
| POST | `/calendar/sync` | Sync all calendars |
| GET | `/users/:user_unique_id/busy_blocks` | List busy blocks |
| POST | `/users/:user_unique_id/busy_blocks` | Create busy block |
| DELETE | `/users/:user_unique_id/busy_blocks/:id` | Delete busy block |
| GET | `/users/:user_unique_id/ics_tokens` | List ICS tokens |
| POST | `/users/:user_unique_id/ics_tokens` | Create ICS token |
| DELETE | `/users/:user_unique_id/ics_tokens/:id` | Delete ICS token |
| GET | `/calendars/:company_url_id/ics/:token` | Public ICS feed (no auth) |
| POST | `/calcom/:url_id/webhook` | Cal.com webhook handler |

---

## Data Models

### CalendarAccount
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `provider` | string | Calendar provider (google, outlook, apple) |
| `email` | string | Calendar account email |
| `sync_enabled` | boolean | Whether sync is enabled |
| `last_synced_at` | timestamp | Last successful sync time |
| `status` | enum | active, inactive, error |
| `created_at` | timestamp | Creation time |

### BusyBlock
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Busy block title |
| `start_time` | timestamp | Start time |
| `end_time` | timestamp | End time |
| `recurring` | boolean | Whether the block recurs |
| `recurrence_rule` | string | RFC 5545 recurrence rule |

### ICSToken
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `token` | string | ICS feed access token |
| `label` | string | Token label |
| `active` | boolean | Whether the token is active |
| `created_at` | timestamp | Creation time |

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Calendar Account Not Found",
    "detail": "The requested calendar account could not be found."
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
// CalendarSyncService — client.crm.calendarSync
syncUser(userUniqueId: string): Promise<CalendarSyncResult>;
syncTenant(): Promise<CalendarSyncResult>;
```

### TypeScript Types

```typescript
import type {
  CalendarSyncResult,
} from '@23blocks/block-crm';
```

### React Hook

```typescript
import { useCrmBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCrmBlock();

  // Example: Sync calendar for a specific user
  const result = await client.crm.calendarSync.syncUser('user-unique-id');
}
```
