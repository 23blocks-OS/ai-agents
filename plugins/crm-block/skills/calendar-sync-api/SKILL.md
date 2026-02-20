---
name: crm-calendar-sync-api
description: Manage 23blocks CRM calendar sync via REST API. Use when connecting calendar accounts, syncing calendars, managing busy blocks, generating ICS tokens, serving public ICS feeds, and handling Cal.com webhooks.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# CRM Calendar Sync API

Complete API reference for 23blocks CRM calendar synchronization with external providers, busy blocks, ICS feeds, and Cal.com integration.

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
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/calendar_accounts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Calendar Account Endpoints

### GET /users/:user_unique_id/calendar_accounts - List Calendar Accounts

Lists all calendar accounts for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/calendar_accounts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "cal-account-uuid-123",
      "type": "calendar_account",
      "attributes": {
        "unique_id": "cal-account-uuid-123",
        "provider": "google",
        "email": "user@gmail.com",
        "sync_enabled": true,
        "last_synced_at": "2026-02-15T10:00:00Z",
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /users/:user_unique_id/calendar_accounts/:id - Get Calendar Account

Retrieves a specific calendar account.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/calendar_accounts/cal-account-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "cal-account-uuid-123",
    "type": "calendar_account",
    "attributes": {
      "unique_id": "cal-account-uuid-123",
      "provider": "google",
      "email": "user@gmail.com",
      "sync_enabled": true,
      "last_synced_at": "2026-02-15T10:00:00Z",
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Calendar account not found

---

### POST /users/:user_unique_id/calendar_accounts - Create Calendar Account

Creates a new calendar account connection.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/calendar_accounts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "calendar_account": {
      "provider": "google",
      "email": "user@gmail.com",
      "sync_enabled": true
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `provider` | string | Yes | Calendar provider (google, outlook, apple) |
| `email` | string | Yes | Calendar account email |
| `sync_enabled` | boolean | No | Enable sync (default: true) |

**Response 201:**
```json
{
  "data": {
    "id": "cal-account-uuid-456",
    "type": "calendar_account",
    "attributes": {
      "unique_id": "cal-account-uuid-456",
      "provider": "google",
      "email": "user@gmail.com",
      "sync_enabled": true,
      "last_synced_at": null,
      "status": "active",
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /users/:user_unique_id/calendar_accounts/:id - Update Calendar Account

Updates a calendar account connection.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/calendar_accounts/cal-account-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "calendar_account": {
      "sync_enabled": false
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "cal-account-uuid-123",
    "type": "calendar_account",
    "attributes": {
      "unique_id": "cal-account-uuid-123",
      "sync_enabled": false,
      "updated_at": "2026-02-15T14:00:00Z"
    }
  }
}
```

---

### DELETE /users/:user_unique_id/calendar_accounts/:id - Delete Calendar Account

Deletes a calendar account connection.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/user-uuid-123/calendar_accounts/cal-account-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Calendar Sync

### POST /users/:user_unique_id/calendar/sync - Sync User Calendar

Triggers a calendar sync for a specific user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/calendar/sync" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Calendar sync initiated",
  "data": {
    "sync_id": "sync-uuid-123",
    "status": "in_progress",
    "started_at": "2026-02-15T10:30:00Z"
  }
}
```

---

### POST /calendar/sync - Sync All Calendars

Triggers a calendar sync for all users.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/calendar/sync" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Global calendar sync initiated",
  "data": {
    "sync_id": "sync-uuid-456",
    "accounts_count": 25,
    "status": "in_progress",
    "started_at": "2026-02-15T10:30:00Z"
  }
}
```

---

## Busy Blocks

### GET /users/:user_unique_id/busy_blocks - List Busy Blocks

Lists all busy time blocks for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/busy_blocks" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "busy-block-uuid-123",
      "type": "busy_block",
      "attributes": {
        "unique_id": "busy-block-uuid-123",
        "title": "Lunch Break",
        "start_time": "2026-02-20T12:00:00Z",
        "end_time": "2026-02-20T13:00:00Z",
        "recurring": true,
        "recurrence_rule": "FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR",
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /users/:user_unique_id/busy_blocks - Create Busy Block

Creates a new busy time block for a user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/busy_blocks" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "busy_block": {
      "title": "Personal Time",
      "start_time": "2026-03-01T17:00:00Z",
      "end_time": "2026-03-01T18:00:00Z",
      "recurring": false
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | No | Busy block title |
| `start_time` | timestamp | Yes | Start time (ISO 8601) |
| `end_time` | timestamp | Yes | End time (ISO 8601) |
| `recurring` | boolean | No | Whether the block recurs (default: false) |
| `recurrence_rule` | string | No | RFC 5545 recurrence rule |

**Response 201:**
```json
{
  "data": {
    "id": "busy-block-uuid-456",
    "type": "busy_block",
    "attributes": {
      "unique_id": "busy-block-uuid-456",
      "title": "Personal Time",
      "start_time": "2026-03-01T17:00:00Z",
      "end_time": "2026-03-01T18:00:00Z",
      "recurring": false,
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

---

### DELETE /users/:user_unique_id/busy_blocks/:id - Delete Busy Block

Deletes a busy time block.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/user-uuid-123/busy_blocks/busy-block-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## ICS Tokens

### GET /users/:user_unique_id/ics_tokens - List ICS Tokens

Lists all ICS feed tokens for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/ics_tokens" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "ics-token-uuid-123",
      "type": "ics_token",
      "attributes": {
        "unique_id": "ics-token-uuid-123",
        "token": "abc123def456ghi789",
        "label": "Personal Calendar Feed",
        "active": true,
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /users/:user_unique_id/ics_tokens - Create ICS Token

Generates a new ICS feed token.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/ics_tokens" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "ics_token": {
      "label": "Shared Team Calendar"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `label` | string | No | Token label for identification |

**Response 201:**
```json
{
  "data": {
    "id": "ics-token-uuid-456",
    "type": "ics_token",
    "attributes": {
      "unique_id": "ics-token-uuid-456",
      "token": "xyz789abc012def345",
      "label": "Shared Team Calendar",
      "active": true,
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

---

### DELETE /users/:user_unique_id/ics_tokens/:id - Delete ICS Token

Deletes an ICS feed token.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/user-uuid-123/ics_tokens/ics-token-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Public ICS Feed

### GET /calendars/:company_url_id/ics/:token - Public ICS Feed

Retrieves a public ICS calendar feed. This endpoint does not require authentication.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/calendars/my-company/ics/abc123def456ghi789"
```

**Response 200:**
Returns an ICS calendar file (text/calendar content type).

```
BEGIN:VCALENDAR
VERSION:2.0
PRODID:-//23blocks//CRM//EN
BEGIN:VEVENT
DTSTART:20260315T140000Z
DTEND:20260315T150000Z
SUMMARY:Quarterly Review
LOCATION:Conference Room A
END:VEVENT
END:VCALENDAR
```

**Errors:**
- `404 Not Found` - Invalid token or company URL ID

---

## Cal.com Integration

### POST /calcom/:url_id/webhook - Cal.com Webhook

Handles incoming Cal.com webhook events for meeting creation and updates.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/calcom/my-company/webhook" \
  -H "Content-Type: application/json" \
  -d '{
    "triggerEvent": "BOOKING_CREATED",
    "payload": {
      "title": "Consultation Call",
      "startTime": "2026-03-20T10:00:00Z",
      "endTime": "2026-03-20T10:30:00Z",
      "attendees": [
        {
          "email": "client@example.com",
          "name": "Client Name"
        }
      ]
    }
  }'
```

**Response 200:**
```json
{
  "message": "Webhook processed successfully"
}
```

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

---

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
