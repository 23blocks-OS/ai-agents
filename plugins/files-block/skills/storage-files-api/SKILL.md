---
name: files-storage-api
description: Manage company-level storage files via REST API. Use for tenant-wide file storage with public distribution and CDN integration.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Storage Files API

Complete API reference for 23blocks company-level storage file management.

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
curl -X GET "$BLOCKS_API_URL/storage/{url_id}/files" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /storage/:url_id/files - List Storage Files

Lists storage files for a company. Admins see all files; others see only public files.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/storage/$URL_ID/files?page=1&records=20&search=logo" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
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

**Note:** No authentication required for public files; AppId header still needed.

---

### PUT /storage/:url_id/presign_upload - Get Presigned URL

Gets a presigned URL for direct S3 upload to storage.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/storage/$URL_ID/presign_upload?filename=banner.jpg&serialization=jsonapi" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filename` | string | Yes | Name of file to upload |
| `serialization` | string | No | Set to `jsonapi` for structured response |

**Response 200 (JSONAPI):**
```json
{
  "data": {
    "type": "presigned_urls",
    "id": 1,
    "attributes": {
      "signed_url": "https://s3.us-east-2.amazonaws.com/...?X-Amz-Signature=...",
      "public_url": "https://s3.us-east-2.amazonaws.com/.../banner.jpg",
      "file_id": "banner.jpg",
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
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "name": "uuid-banner.jpg",
      "original_name": "banner.jpg",
      "url": "https://s3.us-east-2.amazonaws.com/.../uuid-banner.jpg",
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
| `name` | string | Yes | Generated filename |
| `original_name` | string | Yes | Original filename |
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
      "name": "uuid-banner.jpg",
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
  -H "AppId: $BLOCKS_API_KEY" \
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
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /storage/:url_id/files/:unique_file_id/approve - Approve File

Sets file status to active.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/storage/$URL_ID/files/$FILE_ID/approve" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /storage/:url_id/files/:unique_file_id/unpublish - Unpublish File

Removes file from public bucket.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/storage/$URL_ID/files/$FILE_ID/unpublish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### StorageFile
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Generated filename |
| `original_name` | string | Original filename |
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
