---
name: 23blocks-files-user-files-api
description: Upload and manage user files via REST API. Use when uploading files, managing presigned URLs, handling multipart uploads, or processing file workflow operations.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# User Files API

Complete API reference for 23blocks user file management.

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
| GET | `/users/:unique_id/files` | List user files |
| GET | `/users/:unique_id/files/:unique_file_id` | Get file |
| PUT | `/users/:unique_id/presign_upload` | Get presigned URL |
| POST | `/users/:unique_id/multipart_presign_upload` | Start multipart upload |
| POST | `/users/:unique_id/multipart_complete_upload` | Complete multipart upload |
| POST | `/users/:unique_id/files` | Create file |
| PUT | `/users/:unique_id/files/:unique_file_id` | Update file |
| DELETE | `/users/:unique_id/files/:unique_file_id` | Delete file |
| PUT | `/users/:unique_id/files/:unique_file_id/approve` | Approve file |
| PUT | `/users/:unique_id/files/:unique_file_id/reject` | Reject file |
| PUT | `/users/:unique_id/files/:unique_file_id/publish` | Publish file |
| PUT | `/users/:unique_id/files/:unique_file_id/unpublish` | Unpublish file |

---

## Data Models

### UserFile
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | Owner user ID |
| `name` | string | Generated filename |
| `original_name` | string | Original filename |
| `url` | string | S3 URL (signed for private files) |
| `thumbnail_url` | string | Thumbnail URL |
| `file_type` | string | MIME type |
| `file_size` | integer | Size in bytes |
| `description` | string | Description |
| `status` | enum | review, active, deleted |
| `is_public` | boolean | Published to public bucket |
| `access_level` | enum | private, public |
| `category_unique_id` | uuid | Category UUID |
| `tags` | array | Tag names |
| `payload` | json | Custom metadata |
| `ai_enabled` | boolean | RAG processing enabled |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### File Status
| Status | Description |
|--------|-------------|
| `review` | Pending approval |
| `active` | Approved and accessible |
| `deleted` | Soft-deleted |

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
// UserFilesService — client.files.userFiles
list(userUniqueId: string, params?: ListUserFilesParams): Promise<PageResult<UserFile>>;
get(userUniqueId: string, fileUniqueId: string): Promise<UserFile>;
add(userUniqueId: string, data: AddUserFileRequest): Promise<UserFile>;
update(userUniqueId: string, fileUniqueId: string, data: UpdateUserFileRequest): Promise<UserFile>;
delete(userUniqueId: string, fileUniqueId: string): Promise<void>;
presignUpload(userUniqueId: string, data: PresignUploadRequest): Promise<PresignUploadResponse>;
multipartPresign(userUniqueId: string, data: MultipartPresignRequest): Promise<MultipartPresignResponse>;
multipartComplete(userUniqueId: string, data: MultipartCompleteRequest): Promise<UserFile>;
approve(userUniqueId: string, fileUniqueId: string): Promise<UserFile>;
reject(userUniqueId: string, fileUniqueId: string): Promise<UserFile>;
publish(userUniqueId: string, fileUniqueId: string): Promise<UserFile>;
unpublish(userUniqueId: string, fileUniqueId: string): Promise<UserFile>;
```

### TypeScript Types

```typescript
import type {
  UserFile,
  ListUserFilesParams,
  AddUserFileRequest,
  UpdateUserFileRequest,
  PresignUploadRequest,
  PresignUploadResponse,
  MultipartPresignRequest,
  MultipartPresignResponse,
  MultipartCompleteRequest,
} from '@23blocks/block-files';
```

### React Hook

```typescript
import { useFilesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFilesBlock();
  const result = await client.files.userFiles.list('user-unique-id');
}
```
