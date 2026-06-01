---
name: 23blocks-files-storage-api
description: Manage company-level storage files via REST API. Use for tenant-wide file storage with public distribution and CDN integration.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Storage Files API

Complete API reference for 23blocks company-level storage file management.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Files API base URL | `https://files.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

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

## CRITICAL: File Upload Process (Production Incident Fix)

The Files API generates UUID-based S3 keys during presign. You **MUST** use the `file_name` returned from presign endpoints as the `name` field when registering file metadata. Using any other value (such as the original filename) causes **404 errors on download** because the S3 object key will not match.

**Correct upload flow:**
1. `PUT /storage/:url_id/presign_upload?filename=banner.jpg` returns `{ "file_name": "dcca6ec1-...jpg", "signed_url": "..." }`
2. `PUT {signed_url}` with file bytes
3. `POST /storage/:url_id/files` with `{ "name": "dcca6ec1-...jpg", "original_name": "banner.jpg" }` -- `name` **MUST** match `file_name` from step 1

**Common mistake (causes 404 on download):**
```
PUT /presign_upload?filename=banner.jpg  -->  { "file_name": "dcca6ec1-...jpg" }
POST /files with { "name": "banner.jpg" }  <-- WRONG! S3 key mismatch = 404
```

**Field meanings:**
| Field | Purpose | Example |
|-------|---------|---------|
| `name` | S3 object key (UUID-based). Used for downloads. NOT user-facing. | `dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.jpg` |
| `original_name` | User's original filename. Used for display only. | `banner.jpg` |

---

## Endpoints

### GET /storage/:url_id/files - List Storage Files

Lists storage files for a company. Admins see all files; others see only public files.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/storage/$URL_ID/files?page=1&records=20&search=logo" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `search` | string | No | Search by filename |
| `ext` | string | No | Filter by file extension |

**Response 200:**
```json
{
  "data": [
    {
      "id": "storage-file-id",
      "type": "storage_file",
      "attributes": {
        "unique_id": "storage-file-id",
        "name": "company-logo.png",
        "original_name": "logo.png",
        "url": "https://s3.us-east-2.amazonaws.com/...",
        "thumbnail_url": "https://s3.us-east-2.amazonaws.com/...",
        "file_type": "image/png",
        "file_size": 50000,
        "status": "active",
        "is_public": true,
        "created_at": "2025-01-10T10:30:00Z"
      },
      "relationships": {
        "category": {
          "data": { "id": "cat-123", "type": "category" }
        }
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 45
  },
  "links": {
    "self": "/storage/?search=&order=ASC&page=1&size=20"
  }
}
```

**Required Scopes:** `storage:admin` for all files, none for public files only

---

### GET /storage/:url_id/files/:unique_file_id - Get Storage File

Retrieves a single storage file with a fresh signed URL.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/storage/$URL_ID/files/$FILE_ID" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "storage-file-id",
    "type": "storage_file",
    "attributes": {
      "unique_id": "storage-file-id",
      "name": "company-logo.png",
      "original_name": "logo.png",
      "url": "https://s3.us-east-2.amazonaws.com/...?X-Amz-Signature=...",
      "thumbnail_url": "https://s3.us-east-2.amazonaws.com/...",
      "file_type": "image/png",
      "file_size": 50000,
      "description": "Official company logo",
      "status": "active",
      "is_public": true,
      "tags": ["branding", "logo"],
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Note:** No authentication required for public files; X-API-KEY header still needed.

---

### PUT /storage/:url_id/presign_upload - Get Presigned URL

Gets a presigned URL for direct S3 upload to storage.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/storage/$URL_ID/presign_upload?filename=banner.jpg" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filename` | string | Yes | Name of file to upload |
| `serialization` | string | No | Set to `jsonapi` for JSON:API response format |

**Response 200 (default — flat JSON):**
```json
{
  "file_name": "dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.jpg",
  "signed_url": "https://s3.us-east-2.amazonaws.com/...?X-Amz-Signature=...",
  "public_url": "https://s3.us-east-2.amazonaws.com/.../dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.jpg"
}
```

> **CRITICAL:** Save `file_name` from this response. You MUST use it as the `name` field in `POST /files`.

**Response 200 (with `?serialization=jsonapi`):**
```json
{
  "data": {
    "type": "presigned_urls",
    "id": 1,
    "attributes": {
      "file_name": "dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.jpg",
      "signed_url": "https://s3.us-east-2.amazonaws.com/...?X-Amz-Signature=...",
      "public_url": "https://s3.us-east-2.amazonaws.com/.../dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.jpg",
      "file_id": "dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.jpg",
      "expires_at": "2025-01-10T11:30:00Z"
    }
  }
}
```

**Required Scopes:** `storage:write`

---

### POST /storage/:url_id/files - Create Storage File

Registers an uploaded file in storage.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/storage/$URL_ID/files" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "name": "dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.jpg",
      "original_name": "banner.jpg",
      "url": "https://s3.us-east-2.amazonaws.com/.../dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.jpg",
      "file_type": "image/jpeg",
      "file_size": 250000,
      "description": "Homepage banner",
      "category_unique_id": "cat-123",
      "tags": "[\"marketing\", \"homepage\"]",
      "is_public": true
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | **MUST be `file_name` from presign response** (UUID-based S3 key). Using any other value causes 404 on download. |
| `original_name` | string | Yes | Original filename (display only) |
| `url` | string | Yes | S3 URL from presigned upload |
| `file_type` | string | Yes | MIME type |
| `file_size` | integer | Yes | Size in bytes |
| `description` | string | No | File description |
| `category_unique_id` | string | No | Category ID |
| `tags` | string | No | JSON array of tags |
| `is_public` | boolean | No | Publish immediately |
| `ai_enabled` | boolean | No | Enable AI/RAG processing |
| `is_temp` | boolean | No | Mark as temporary file |
| `payload` | string | No | Custom JSON payload |

**Response 201:**
```json
{
  "data": {
    "id": "new-storage-file-id",
    "type": "storage_file",
    "attributes": {
      "unique_id": "new-storage-file-id",
      "name": "dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.jpg",
      "original_name": "banner.jpg",
      "status": "review",
      "is_public": true,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### PUT /storage/:url_id/files/:unique_file_id - Update Storage File

Updates storage file metadata.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/storage/$URL_ID/files/$FILE_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "description": "Updated banner description",
      "tags": "[\"marketing\", \"homepage\", \"2025\"]"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "storage-file-id",
    "type": "storage_file",
    "attributes": {
      "description": "Updated banner description",
      "updated_at": "2025-01-10T14:00:00Z"
    }
  }
}
```

---

### DELETE /storage/:url_id/files/:unique_file_id - Delete Storage File

Soft-deletes a storage file.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/storage/$URL_ID/files/$FILE_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /storage/:url_id/files/:unique_file_id/approve - Approve File

Sets file status to active.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/storage/$URL_ID/files/$FILE_ID/approve" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "storage-file-id",
    "type": "storage_file",
    "attributes": {
      "status": "active"
    }
  }
}
```

---

### PUT /storage/:url_id/files/:unique_file_id/reject - Reject File

Sets file status back to review.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/storage/$URL_ID/files/$FILE_ID/reject" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "storage-file-id",
    "type": "storage_file",
    "attributes": {
      "status": "review"
    }
  }
}
```

---

### PUT /storage/:url_id/files/:unique_file_id/publish - Publish File

Copies file to public bucket for public access.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/storage/$URL_ID/files/$FILE_ID/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /storage/:url_id/files/:unique_file_id/unpublish - Unpublish File

Removes file from public bucket.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/storage/$URL_ID/files/$FILE_ID/unpublish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

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

---

## Use Cases

### Public Website Assets
Storage files are ideal for company-wide assets like:
- Logos and branding images
- Marketing banners
- Downloadable PDFs
- Podcast episodes (with RSS feed support)

### CDN Distribution
For high-traffic assets, configure CloudFront for CDN distribution. The public URL can be transformed to use CloudFront when configured.

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Failed",
    "detail": "Name can't be blank."
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

### React Hook

```typescript
import { useFilesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFilesBlock();
  const result = await client.files.storageFiles.list();
}
```
