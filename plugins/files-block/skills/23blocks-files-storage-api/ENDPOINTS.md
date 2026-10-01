# Storage Files API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

## Contents

- GET `/storage/:url_id/files`: List Storage Files
- GET `/storage/:url_id/files/:unique_file_id`: Get Storage File
- PUT `/storage/:url_id/presign_upload`: Get Presigned URL
- POST `/storage/:url_id/files`: Create Storage File
- PUT `/storage/:url_id/files/:unique_file_id`: Update Storage File
- DELETE `/storage/:url_id/files/:unique_file_id`: Delete Storage File
- PUT `/storage/:url_id/files/:unique_file_id/approve`: Approve File
- PUT `/storage/:url_id/files/:unique_file_id/reject`: Reject File
- PUT `/storage/:url_id/files/:unique_file_id/publish`: Publish File
- PUT `/storage/:url_id/files/:unique_file_id/unpublish`: Unpublish File

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

> Keep `file_name` from this response: it is the `name` field for `POST /files`.

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
| `name` | string | Yes | The `file_name` from the presign response (UUID-based S3 key). Any other value causes 404 on download. |
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
