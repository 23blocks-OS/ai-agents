---
name: 23blocks-crm-accounts-api
description: "CRM Block customer accounts: CRUD, archive, details, logo, documents, related leads, contacts, meetings, quotes. Use for company records."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Accounts API

Complete API reference for 23blocks CRM account management with details, documents, leads, contacts, meetings, locations, categories, and quotes.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://crm.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/accounts/` | List accounts (paginated) |
| GET | `/accounts/:unique_id` | Get account by ID |
| GET | `/accounts/status/trash` | List trashed accounts |
| GET | `/accounts/status/archive` | List archived accounts |
| POST | `/accounts/validate` | Validate account data |
| POST | `/accounts/validate_tax_id` | Validate tax ID |
| POST | `/accounts/` | Create account |
| PUT | `/accounts/:unique_id/` | Update account |
| DELETE | `/accounts/:unique_id/` | Delete account (soft) |
| DELETE | `/accounts/:unique_id/archive` | Archive account |
| POST | `/accounts/:unique_id/details` | Add account detail |
| PUT | `/accounts/:unique_id/details` | Update account details |
| PUT | `/accounts/:unique_id/presign_logo` | Get logo upload URL |
| PUT | `/accounts/:unique_id/logo/publish` | Publish account logo |
| GET | `/accounts/:unique_id/leads` | List account leads |
| POST | `/accounts/:unique_id/leads` | Add lead to account |
| GET | `/accounts/:unique_id/contacts` | List account contacts |
| GET | `/accounts/:unique_id/contacts/:contact_unique_id` | Get specific contact |
| POST | `/accounts/:unique_id/contacts` | Add contact to account |
| DELETE | `/accounts/:unique_id/contacts/:contact_unique_id` | Remove contact |
| GET | `/accounts/:unique_id/meetings` | List account meetings |
| GET | `/accounts/:unique_id/documents` | List documents |
| PUT | `/accounts/:unique_id/presign_document` | Get document upload URL |
| POST | `/accounts/:unique_id/documents` | Upload document |
| DELETE | `/accounts/:unique_id/documents/:unique_document_id` | Delete document |
| POST | `/accounts/:unique_id/categories` | Assign category |
| GET | `/accounts/:unique_id/locations` | List locations |
| POST | `/accounts/:unique_id/locations` | Add location |
| GET | `/accounts/:unique_id/quotes` | List quotes |
| POST | `/accounts/:unique_id/quotes` | Add quote |

---

## Data Models

### Account
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Account name |
| `tax_id` | string | Tax identification number |
| `industry` | string | Industry sector |
| `website` | string | Website URL |
| `phone` | string | Phone number |
| `email` | string | Email address |
| `address` | string | Physical address |
| `status` | enum | active, archived, trash |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Location
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Location name |
| `address` | string | Street address |
| `city` | string | City |
| `state` | string | State/Province |
| `zip` | string | ZIP/Postal code |
| `country` | string | Country code |

### Document
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `filename` | string | File name |
| `content_type` | string | MIME type |
| `file_size` | integer | File size in bytes |
| `url` | string | Download URL |
| `created_at` | timestamp | Upload time |

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Account Not Found","detail":"The requested account could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-crm`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCrmBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// AccountsService — client.crm.accounts
list(params?: ListAccountsParams): Promise<PageResult<Account>>;
get(uniqueId: string): Promise<Account>;
create(data: CreateAccountRequest): Promise<Account>;
update(uniqueId: string, data: UpdateAccountRequest): Promise<Account>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<Account>;
search(query: string, params?: ListAccountsParams): Promise<PageResult<Account>>;
listDeleted(params?: ListAccountsParams): Promise<PageResult<Account>>;
```

### TypeScript Types

```typescript
import type {
  Account,
  AccountDetail,
  CreateAccountRequest,
  UpdateAccountRequest,
  ListAccountsParams,
} from '@23blocks/block-crm';
```
