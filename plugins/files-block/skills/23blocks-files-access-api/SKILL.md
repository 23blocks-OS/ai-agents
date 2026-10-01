---
name: 23blocks-files-access-api
description: "Files Block access to a user's file: grant, revoke, public or private, access requests, bulk. Use when sharing files."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# File Access Control API

Complete API reference for 23blocks file access control management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://files.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users/:unique_id/files/:unique_file_id/access` | Get file access list |
| POST | `/users/:unique_id/files/:unique_file_id/access/grant` | Grant access |
| DELETE | `/users/:unique_id/files/:unique_file_id/access/:access_unique_id/revoke` | Revoke access |
| POST | `/users/:unique_id/files/:unique_file_id/access/make_public` | Make file public |
| POST | `/users/:unique_id/files/:unique_file_id/access/make_private` | Make file private |
| POST | `/users/:unique_id/files/access/grant` | Bulk grant access |
| POST | `/users/:unique_id/files/access/revoke` | Bulk revoke access |
| POST | `/users/:unique_id/files/access/grant-to-users` | Grant to multiple users |
| POST | `/users/:unique_id/files/access/revoke-from-users` | Revoke from multiple users |
| GET | `/users/:unique_id/access/summary` | Access summary for user |
| POST | `/users/:unique_id/files/:unique_file_id/requests/access` | Request access |
| GET | `/users/:unique_id/files/:unique_file_id/access/requests` | List access requests |
| PUT | `/users/:unique_id/files/:unique_file_id/access/requests/:request_unique_id/approve` | Approve request |
| DELETE | `/users/:unique_id/files/:unique_file_id/access/requests/:request_unique_id/deny` | Deny request |

---

## Data Models

### UserFileAccess
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_file_unique_id` | uuid | File ID |
| `user_unique_id` | uuid | User with access |
| `access_type` | enum | owner, write, read |
| `starts_at` | timestamp | When access starts |
| `expires_at` | timestamp | When access expires (null = never) |
| `created_at` | timestamp | Creation time |

### UserFileAccessRequest
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_file_unique_id` | uuid | File ID |
| `user_unique_id` | uuid | Requesting user |
| `access_type` | enum | Requested access level |
| `status` | enum | pending, approved, denied |
| `requested_by_user_name` | string | Requester name |
| `requested_by_email` | string | Requester email |
| `approved_by_user_name` | string | Approver name |
| `approved_at` | timestamp | Approval time |
| `created_at` | timestamp | Request time |

### Access Types
| Type | Description |
|------|-------------|
| `owner` | Full control (create, read, update, delete) |
| `write` | Can modify file and metadata |
| `read` | View-only access |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"403","source":"Files Management","code":"uf832069","title":"Forbidden","detail":"Insufficient Scope"}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-files`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useFilesBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// FileAccessService — client.files.fileAccess
list(params?: ListFileAccessParams): Promise<PageResult<FileAccess>>;
get(uniqueId: string): Promise<FileAccess>;
grant(data: CreateFileAccessRequest): Promise<FileAccess>;
update(uniqueId: string, data: UpdateFileAccessRequest): Promise<FileAccess>;
revoke(uniqueId: string): Promise<void>;
listByFile(fileUniqueId: string, params?: ListFileAccessParams): Promise<PageResult<FileAccess>>;
listByGrantee(granteeUniqueId: string, granteeType: string, params?: ListFileAccessParams): Promise<PageResult<FileAccess>>;
checkAccess(fileUniqueId: string, granteeUniqueId: string): Promise<FileAccess | null>;
```

### TypeScript Types

```typescript
import type {
  FileAccess,
  CreateFileAccessRequest,
  UpdateFileAccessRequest,
  ListFileAccessParams,
} from '@23blocks/block-files';
```
