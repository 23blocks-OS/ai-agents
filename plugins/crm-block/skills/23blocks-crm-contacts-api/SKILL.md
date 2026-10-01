---
name: 23blocks-crm-contacts-api
description: "CRM Block contacts: CRUD, archive, profile, history, documents. Use for people records."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Contacts API

Complete API reference for 23blocks CRM contact management with profiles, history tracking, and document handling.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://crm.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Contact Not Found","detail":"The requested contact could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-crm`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCrmBlock()` from `@23blocks/react`.

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
