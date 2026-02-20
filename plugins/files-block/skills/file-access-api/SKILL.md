---
name: files-access-api
description: Manage file access control via REST API. Grant, revoke, and manage file permissions. Handle access requests with approval workflows. Supports bulk operations for managing access across multiple files and users.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# File Access Control API

Complete API reference for 23blocks file access control management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://files.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Files API base URL | `https://files.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/users/{user_id}/files/{file_id}/access" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Access Control Endpoints

### GET /users/:unique_id/files/:unique_file_id/access - Get File Access

Retrieves the access list for a file.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "access-id-1",
      "type": "user_file_access",
      "attributes": {
        "unique_id": "access-id-1",
        "user_unique_id": "user-123",
        "access_type": "owner",
        "starts_at": "2025-01-10T10:30:00Z",
        "expires_at": null,
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "access-id-2",
      "type": "user_file_access",
      "attributes": {
        "unique_id": "access-id-2",
        "user_unique_id": "user-456",
        "access_type": "read",
        "starts_at": "2025-01-12T09:00:00Z",
        "expires_at": "2025-06-12T09:00:00Z",
        "created_at": "2025-01-12T09:00:00Z"
      }
    }
  ]
}
```

---

### POST /users/:unique_id/files/:unique_file_id/access/grant - Grant Access

Grants access to a file for another user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/grant" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access": {
      "user_unique_id": "target-user-id",
      "access_type": "read",
      "expires_at": "2025-12-31T23:59:59Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | string | Yes | User to grant access to |
| `access_type` | enum | Yes | owner, write, read |
| `expires_at` | timestamp | No | When access expires |

**Response 201:**
```json
{
  "data": {
    "id": "new-access-id",
    "type": "user_file_access",
    "attributes": {
      "unique_id": "new-access-id",
      "user_unique_id": "target-user-id",
      "access_type": "read",
      "starts_at": "2025-01-10T10:30:00Z",
      "expires_at": "2025-12-31T23:59:59Z"
    }
  }
}
```

---

### DELETE /users/:unique_id/files/:unique_file_id/access/:access_unique_id/revoke - Revoke Access

Revokes a specific access grant.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/$ACCESS_ID/revoke" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### POST /users/:unique_id/files/:unique_file_id/access/make_public - Make File Public

Changes file access level to public.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/make_public" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "file-id",
    "type": "user_file",
    "attributes": {
      "access_level": "public"
    }
  }
}
```

---

### POST /users/:unique_id/files/:unique_file_id/access/make_private - Make File Private

Changes file access level to private.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/make_private" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "file-id",
    "type": "user_file",
    "attributes": {
      "access_level": "private"
    }
  }
}
```

---

## Bulk Operations

### POST /users/:unique_id/files/access/grant - Bulk Grant Access

Grants a user access to multiple files at once.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/access/grant" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access": {
      "user_unique_id": "target-user-id",
      "access_type": "read",
      "file_unique_ids": ["file-1", "file-2", "file-3"]
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | string | Yes | User to grant access to |
| `access_type` | enum | Yes | owner, write, read |
| `file_unique_ids` | array | Yes | List of file IDs |

**Response 200:**
```json
{
  "data": {
    "granted": 3,
    "failed": 0,
    "results": [
      {"file_id": "file-1", "status": "granted"},
      {"file_id": "file-2", "status": "granted"},
      {"file_id": "file-3", "status": "granted"}
    ]
  }
}
```

---

### POST /users/:unique_id/files/access/revoke - Bulk Revoke Access

Revokes a user's access from multiple files.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/access/revoke" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access": {
      "user_unique_id": "target-user-id",
      "file_unique_ids": ["file-1", "file-2", "file-3"]
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "revoked": 3,
    "failed": 0
  }
}
```

---

### POST /users/:unique_id/files/access/grant-to-users - Grant Access to Multiple Users

Grants multiple users access to all of a user's files.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/access/grant-to-users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access": {
      "user_unique_ids": ["user-1", "user-2", "user-3"],
      "access_type": "read"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_ids` | array | Yes | Users to grant access to |
| `access_type` | enum | Yes | owner, write, read |

**Response 200:**
```json
{
  "data": {
    "users_granted": 3,
    "files_affected": 25
  }
}
```

---

### POST /users/:unique_id/files/access/revoke-from-users - Revoke Access from Multiple Users

Revokes multiple users' access from all of a user's files.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/access/revoke-from-users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access": {
      "user_unique_ids": ["user-1", "user-2", "user-3"]
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "users_revoked": 3,
    "files_affected": 25
  }
}
```

---

## Access Request Endpoints

### POST /users/:unique_id/files/:unique_file_id/requests/access - Request Access

Submits an access request for a file.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/requests/access" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "request": {
      "access_type": "read",
      "reason": "Need access for project review"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `access_type` | enum | Yes | owner, write, read |
| `reason` | string | No | Reason for request |

**Response 201:**
```json
{
  "data": {
    "id": "request-id",
    "type": "user_file_access_request",
    "attributes": {
      "unique_id": "request-id",
      "access_type": "read",
      "status": "pending",
      "requested_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### GET /users/:unique_id/files/:unique_file_id/access/requests - List Access Requests

Lists pending access requests for a file.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/requests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "request-id",
      "type": "user_file_access_request",
      "attributes": {
        "unique_id": "request-id",
        "user_unique_id": "requesting-user-id",
        "access_type": "read",
        "status": "pending",
        "requested_by_user_name": "John Doe",
        "requested_by_email": "john@example.com",
        "requested_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### PUT /users/:unique_id/files/:unique_file_id/access/requests/:request_unique_id/approve - Approve Request

Approves an access request and grants access.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/requests/$REQUEST_ID/approve" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "request-id",
    "type": "user_file_access_request",
    "attributes": {
      "status": "approved",
      "approved_at": "2025-01-10T14:00:00Z",
      "approved_by_user_name": "File Owner"
    }
  }
}
```

---

### DELETE /users/:unique_id/files/:unique_file_id/access/requests/:request_unique_id/deny - Deny Request

Denies an access request.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/requests/$REQUEST_ID/deny" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

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
| `requested_by_ip` | string | Request IP address |
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
