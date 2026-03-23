---
name: 23blocks-files-access-api
description: Manage file access control via REST API. Use when granting or revoking file permissions, handling access requests, or performing bulk access operations.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# File Access Control API

Complete API reference for 23blocks file access control management.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Files API base URL | `https://files.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://files.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://files.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

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

## Error Response Format

```json
{
  "errors": [{
    "status": "403",
    "source": "Files Management",
    "code": "uf832069",
    "title": "Forbidden",
    "detail": "Insufficient Scope"
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-files
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

### React Hook

```typescript
import { useFilesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFilesBlock();
  const result = await client.files.fileAccess.list();
}
```
