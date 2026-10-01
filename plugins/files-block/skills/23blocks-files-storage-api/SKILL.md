---
name: 23blocks-files-storage-api
description: "Files Block company-wide storage with public CDN distribution. Use for tenant assets such as site images, not per-user files."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Storage Files API

Complete API reference for 23blocks company-level storage file management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://files.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## File upload: `name` is the presign `file_name`

The Files API generates UUID-based S3 keys during presign. Use the `file_name` returned by the presign endpoint as the `name` field when registering file metadata. Using any other value (such as the original filename) causes **404 errors on download** because the S3 object key will not match.

**Correct upload flow:**
1. `PUT /storage/:url_id/presign_upload?filename=banner.jpg` returns `{ "file_name": "dcca6ec1-...jpg", "signed_url": "..." }`
2. `PUT {signed_url}` with file bytes
3. `POST /storage/:url_id/files` with `{ "name": "dcca6ec1-...jpg", "original_name": "banner.jpg" }` -- `name` is `file_name` from step 1

**Common mistake (causes 404 on download):**
```
PUT /presign_upload?filename=banner.jpg  -->  { "file_name": "dcca6ec1-...jpg" }
POST /files with { "name": "banner.jpg" }  <-- wrong: S3 key mismatch, 404 on download
```

**Field meanings:**
| Field | Purpose | Example |
|-------|---------|---------|
| `name` | S3 object key (UUID-based). Used for downloads; not user-facing. | `dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.jpg` |
| `original_name` | User's original filename. Used for display only. | `banner.jpg` |

## Endpoints

Request and response detail for each endpoint: [ENDPOINTS.md](ENDPOINTS.md).

| Method | Path | Description |
|--------|------|-------------|
| GET | `/storage/:url_id/files` | List Storage Files |
| GET | `/storage/:url_id/files/:unique_file_id` | Get Storage File |
| PUT | `/storage/:url_id/presign_upload` | Get Presigned URL |
| POST | `/storage/:url_id/files` | Create Storage File |
| PUT | `/storage/:url_id/files/:unique_file_id` | Update Storage File |
| DELETE | `/storage/:url_id/files/:unique_file_id` | Delete Storage File |
| PUT | `/storage/:url_id/files/:unique_file_id/approve` | Approve File |
| PUT | `/storage/:url_id/files/:unique_file_id/reject` | Reject File |
| PUT | `/storage/:url_id/files/:unique_file_id/publish` | Publish File |
| PUT | `/storage/:url_id/files/:unique_file_id/unpublish` | Unpublish File |

## Data Models

### StorageFile
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | UUID-based S3 key (from presign `file_name`). Used for downloads. |
| `original_name` | string | User's original filename (display only) |
| `url` | string | S3 URL |
| `thumbnail_url` | string | Thumbnail URL |
| `file_type` | string | MIME type |
| `file_size` | integer | Size in bytes |
| `description` | string | Description |
| `status` | enum | review, active, deleted, unpublished |
| `is_public` | boolean | Published to public bucket |
| `category_id` | integer | Category FK |
| `category_name` | string | Category name |
| `tags` | array | Tag names |
| `payload` | json | Custom metadata |
| `ai_enabled` | boolean | RAG processing enabled |
| `is_temp` | boolean | Temporary file flag |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

## Use Cases

### Public Website Assets
Storage files are ideal for company-wide assets like:
- Logos and branding images
- Marketing banners
- Downloadable PDFs
- Podcast episodes (with RSS feed support)

### CDN Distribution
For high-traffic assets, configure CloudFront for CDN distribution. The public URL can be transformed to use CloudFront when configured.

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Failed","detail":"Name can't be blank."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-files`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useFilesBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// StorageFilesService — client.files.storageFiles
list(params?: ListStorageFilesParams): Promise<PageResult<StorageFile>>;
get(uniqueId: string): Promise<StorageFile>;
upload(data: UploadFileRequest): Promise<StorageFile>;
create(data: CreateStorageFileRequest): Promise<StorageFile>;
update(uniqueId: string, data: UpdateStorageFileRequest): Promise<StorageFile>;
delete(uniqueId: string): Promise<void>;
download(uniqueId: string): Promise<Blob>;
listByOwner(ownerUniqueId: string, ownerType: string, params?: ListStorageFilesParams): Promise<PageResult<StorageFile>>;
```

### TypeScript Types

```typescript
import type {
  StorageFile,
  CreateStorageFileRequest,
  UpdateStorageFileRequest,
  ListStorageFilesParams,
  UploadFileRequest,
} from '@23blocks/block-files';
```
