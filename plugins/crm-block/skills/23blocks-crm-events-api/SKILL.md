---
name: 23blocks-crm-events-api
description: "CRM Block events: check-in/checkout and confirmation for contacts and employees, admin notes. Use for event attendance."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Events API

Complete API reference for 23blocks CRM event management with participant check-in/checkout workflows.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://crm.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Event Not Found","detail":"The requested event could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-crm`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCrmBlock()` from `@23blocks/react`.

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
