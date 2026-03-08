---
name: 23blocks-crm-contacts-api
description: Manage 23blocks CRM contacts via REST API. Use when creating, updating, archiving contacts, managing contact profiles, history, and documents.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Contacts API

Complete API reference for 23blocks CRM contact management with profiles, history tracking, and document handling.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | CRM API base URL | `https://crm.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/contacts/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/contacts/` | List contacts (paginated) |
| GET | `/contacts/:unique_id` | Get contact by ID |
| GET | `/contacts/trash/show` | List trashed contacts |
| POST | `/contacts/` | Create contact |
| POST | `/contacts/:unique_id/history` | Add history entry |
| PUT | `/contacts/:unique_id/` | Update contact |
| POST | `/contacts/:unique_id/profile` | Add profile |
| PUT | `/contacts/:unique_id/profile` | Update profile |
| DELETE | `/contacts/:unique_id/` | Delete contact (soft) |
| DELETE | `/contacts/:unique_id/archive` | Archive contact |
| GET | `/contacts/:unique_id/documents` | List documents |
| POST | `/contacts/:unique_id/documents` | Upload document |
| DELETE | `/contacts/:unique_id/documents/:unique_document_id` | Delete document |
| PUT | `/contacts/:contact_unique_id/presign_document` | Get document upload URL |

---

## Data Models

### Contact
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `email` | string | Email address |
| `phone` | string | Phone number |
| `account_id` | uuid | Associated account ID |
| `title` | string | Job title |
| `status` | enum | active, archived, trash |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### ContactProfile
| Field | Type | Description |
|-------|------|-------------|
| `bio` | string | Biography |
| `linkedin_url` | string | LinkedIn profile URL |
| `timezone` | string | Timezone |

### ContactHistory
| Field | Type | Description |
|-------|------|-------------|
| `action` | string | Type of history entry |
| `description` | string | Description of interaction |
| `occurred_at` | timestamp | When the interaction occurred |
| `created_at` | timestamp | Record creation time |

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Contact Not Found",
    "detail": "The requested contact could not be found."
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
// ContactsService — client.crm.contacts
list(params?: ListContactsParams): Promise<PageResult<Contact>>;
get(uniqueId: string): Promise<Contact>;
create(data: CreateContactRequest): Promise<Contact>;
update(uniqueId: string, data: UpdateContactRequest): Promise<Contact>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<Contact>;
search(query: string, params?: ListContactsParams): Promise<PageResult<Contact>>;
listDeleted(params?: ListContactsParams): Promise<PageResult<Contact>>;
```

### TypeScript Types

```typescript
import type {
  Contact,
  ContactProfile,
  CreateContactRequest,
  UpdateContactRequest,
  ListContactsParams,
} from '@23blocks/block-crm';
```

### React Hook

```typescript
import { useCrmBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCrmBlock();

  // Example: List all contacts
  const result = await client.crm.contacts.list();
}
```
