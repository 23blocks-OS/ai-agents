---
name: 23blocks-files-user-files-api
description: "Files Block per-user files: presigned and multipart upload, approve, reject, publish. Use when a user uploads or manages their files."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# User Files API

Complete API reference for 23blocks user file management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://files.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## File upload: `name` is the presign `file_name`

The Files API generates UUID-based S3 keys during presign. Use the `file_name` returned by the presign endpoint as the `name` field when registering file metadata. Using any other value (such as the original filename) causes **404 errors on download** because the S3 object key will not match.

**Correct single-file upload flow:**
1. `PUT /users/:id/presign_upload?filename=photo.jpg` returns `{ "file_name": "dcca6ec1-...jpg", "signed_url": "..." }`
2. `PUT {signed_url}` with file bytes
3. `POST /users/:id/files` with `{ "name": "dcca6ec1-...jpg", "original_name": "photo.jpg" }` -- `name` is `file_name` from step 1

**Correct multipart upload flow (large files):**
1. `POST /users/:id/multipart_presign_upload` with `{ "filename": "large.mp4", "part_count": 5 }` returns `{ "file_name": "a1b2c3d4-...mp4", "upload_id": "...", "presigned_urls": [...] }`
2. Upload each part to its presigned URL, collect ETags
3. `POST /users/:id/multipart_complete_upload` with `{ "file_name": "a1b2c3d4-...mp4", "upload_id": "...", "parts": [...] }`
4. `POST /users/:id/files` with `{ "name": "a1b2c3d4-...mp4", "original_name": "large.mp4" }` -- `name` is `file_name` from step 1

**Common mistake (causes 404 on download):**
```
PUT /presign_upload?filename=photo.jpg  -->  { "file_name": "dcca6ec1-...jpg" }
POST /files with { "name": "photo.jpg" }  <-- wrong: S3 key mismatch, 404 on download
```

**Field meanings:**
| Field | Purpose | Example |
|-------|---------|---------|
| `name` | S3 object key (UUID-based). Used for downloads; not user-facing. | `dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.jpg` |
| `original_name` | User's original filename. Used for display only. | `photo.jpg` |

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
| `name` | string | UUID-based S3 key (from presign `file_name`). Used for downloads. |
| `original_name` | string | User's original filename (display only) |
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"403","source":"Files Management","code":"uf832069","title":"Forbidden","detail":"Insufficient Scope"}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-files`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
