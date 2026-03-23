---
name: 23blocks-crm-events-api
description: Manage 23blocks CRM events via REST API. Use when creating events, managing participant check-in/checkout workflows, updating confirmation status for contacts and employees, and handling admin notes.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Events API

Complete API reference for 23blocks CRM event management with participant check-in/checkout workflows.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Crm API base URL | `https://crm.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://crm.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://crm.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/events/` | List events (paginated) |
| POST | `/events/` | Create event |
| PUT | `/events/:unique_id` | Update event |
| DELETE | `/events/:unique_id` | Delete event |
| PUT | `/events/:unique_id/contacts/confirmation` | Update contact confirmation |
| PUT | `/events/:unique_id/contacts/checking` | Record contact check-in |
| PUT | `/events/:unique_id/contacts/checkout` | Record contact checkout |
| PUT | `/events/:unique_id/contacts/notes` | Add/update contact notes |
| PUT | `/events/:unique_id/employees/confirmation` | Update employee confirmation |
| PUT | `/events/:unique_id/employees/checking` | Record employee check-in |
| PUT | `/events/:unique_id/employees/checkout` | Record employee checkout |
| PUT | `/events/:unique_id/admin/notes` | Add/update admin notes |

---

## Data Models

### Event
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Event title |
| `event_type` | string | Event type (conference, webinar, workshop, meetup) |
| `start_time` | timestamp | Start time (ISO 8601) |
| `end_time` | timestamp | End time (ISO 8601) |
| `location` | string | Event location |
| `participants` | integer | Number of participants |
| `status` | enum | scheduled, in_progress, completed, cancelled |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Event Not Found",
    "detail": "The requested event could not be found."
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
// ContactEventsService — client.crm.contactEvents
list(params?: ListContactEventsParams): Promise<PageResult<ContactEvent>>;
get(uniqueId: string): Promise<ContactEvent>;
create(data: CreateContactEventRequest): Promise<ContactEvent>;
update(uniqueId: string, data: UpdateContactEventRequest): Promise<ContactEvent>;
delete(uniqueId: string): Promise<void>;
studentConfirmation(uniqueId: string, request?: ConfirmationRequest): Promise<ContactEvent>;
studentCheckin(uniqueId: string, request?: CheckinRequest): Promise<ContactEvent>;
teacherConfirmation(uniqueId: string, request?: ConfirmationRequest): Promise<ContactEvent>;
teacherCheckin(uniqueId: string, request?: CheckinRequest): Promise<ContactEvent>;
checkout(uniqueId: string, request?: CheckoutRequest): Promise<ContactEvent>;
checkoutStudent(uniqueId: string, request?: CheckoutRequest): Promise<ContactEvent>;
studentNotes(uniqueId: string, request: EventNotesRequest): Promise<ContactEvent>;
adminNotes(uniqueId: string, request: EventNotesRequest): Promise<ContactEvent>;
```

### TypeScript Types

```typescript
import type {
  ContactEvent,
  CreateContactEventRequest,
  UpdateContactEventRequest,
  ListContactEventsParams,
  ConfirmationRequest,
  CheckinRequest,
  CheckoutRequest,
  EventNotesRequest,
} from '@23blocks/block-crm';
```

### React Hook

```typescript
import { useCrmBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCrmBlock();

  // Example: List all contact events
  const result = await client.crm.contactEvents.list();
}
```
