# User Files API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

### GET /users/:unique_id/files - List User Files

Lists all files for a user with pagination and filtering.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/files?page=1&records=20&search=contract" \
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
| `tags` | string | No | Filter by tags |

**Response 200:**
```json
{
  "data": [
    {
      "id": "9ebed093-574a-4be4-9076-cfca36493330",
      "type": "user_file",
      "attributes": {
        "unique_id": "9ebed093-574a-4be4-9076-cfca36493330",
        "name": "abc123-document.pdf",
        "original_name": "contract.pdf",
        "url": "https://s3.us-east-2.amazonaws.com/...",
        "thumbnail_url": "https://s3.us-east-2.amazonaws.com/.../thumbnail_...",
        "file_type": "application/pdf",
        "file_size": 1024000,
        "status": "active",
        "is_public": false,
        "access_level": "private",
        "category_name": "Contracts",
        "tags": ["legal", "2025"],
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

**Required Scopes:** `user_files:read` or `user_files:manage`

---

### GET /users/:unique_id/files/:unique_file_id - Get File

Retrieves a single file with a fresh signed URL.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "9ebed093-574a-4be4-9076-cfca36493330",
    "type": "user_file",
    "attributes": {
      "unique_id": "9ebed093-574a-4be4-9076-cfca36493330",
      "name": "abc123-document.pdf",
      "original_name": "contract.pdf",
      "url": "https://s3.us-east-2.amazonaws.com/...?X-Amz-Signature=...",
      "thumbnail_url": "https://s3.us-east-2.amazonaws.com/...",
      "file_type": "application/pdf",
      "file_size": 1024000,
      "description": "Annual contract 2025",
      "status": "active",
      "is_public": false,
      "access_level": "private",
      "category_name": "Contracts",
      "category_unique_id": "cat-123",
      "tags": ["legal", "2025"],
      "payload": {},
      "ai_enabled": false,
      "created_by": "John Doe",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - File not found or no access

**Required Scopes:** `user_files:read`

---

### PUT /users/:unique_id/presign_upload - Get Presigned URL

Gets a presigned URL for direct S3 upload.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/$USER_ID/presign_upload?filename=document.pdf&serialization=jsonapi" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filename` | string | Yes | Name of file to upload |
| `file_name` | string | No | Alternative to filename (backward compat) |
| `serialization` | string | No | Set to `jsonapi` for structured response |

**Response 200 (JSONAPI):**
```json
{
  "data": {
    "type": "presigned_urls",
    "id": 1,
    "attributes": {
      "signed_url": "https://s3.us-east-2.amazonaws.com/...?X-Amz-Signature=...",
      "public_url": "https://s3.us-east-2.amazonaws.com/.../document.pdf",
      "file_id": "document.pdf",
      "expires_at": "2025-01-10T11:30:00Z"
    }
  }
}
```

**Required Scopes:** `user_files:write`

---

### POST /users/:unique_id/multipart_presign_upload - Start Multipart Upload

Initiates a multipart upload for large files (> 100MB recommended).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/multipart_presign_upload" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "large-video.mp4",
    "part_count": 5
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filename` | string | Yes | Name of file to upload |
| `part_count` | integer | Yes | Number of parts to split file into |
| `serialization` | string | No | Set to `jsonapi` for structured response |

**Response 200 (JSONAPI):**
```json
{
  "data": {
    "type": "presigned_urls",
    "id": 1,
    "attributes": {
      "upload_id": "abc123xyz",
      "presigned_urls": [
        "https://s3...?partNumber=1&uploadId=abc123xyz&X-Amz-Signature=...",
        "https://s3...?partNumber=2&uploadId=abc123xyz&X-Amz-Signature=..."
      ],
      "file_id": "large-video.mp4",
      "expires_at": "2025-01-10T11:30:00Z"
    }
  }
}
```

**Required Scopes:** `user_files:write`

---

### POST /users/:unique_id/multipart_complete_upload - Complete Multipart Upload

Finalizes a multipart upload after all parts are uploaded.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/multipart_complete_upload" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "large-video.mp4",
    "upload_id": "abc123xyz",
    "parts": [
      {"PartNumber": 1, "ETag": "\"abc123\""},
      {"PartNumber": 2, "ETag": "\"def456\""}
    ]
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filename` | string | Yes | Name of file |
| `upload_id` | string | Yes | Upload ID from multipart_presign |
| `parts` | array | Yes | Array of part info with PartNumber and ETag |

**Response 200:**
```json
{
  "data": {
    "type": "presigned_urls",
    "id": 1,
    "attributes": {
      "public_url": "https://s3.us-east-2.amazonaws.com/.../large-video.mp4",
      "file_name": "large-video.mp4",
      "file_id": "large-video.mp4",
      "expires_at": "2025-01-10T11:30:00Z"
    }
  }
}
```

**Errors:**
- `400 uf101421` - Missing parameters
- `400 uf101521` - Invalid parts format

**Required Scopes:** `user_files:write`

---

### POST /users/:unique_id/files - Create File

Registers an uploaded file in the system.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "name": "uuid-generated-name.pdf",
      "original_name": "contract.pdf",
      "url": "https://s3.us-east-2.amazonaws.com/.../uuid-generated-name.pdf",
      "file_type": "application/pdf",
      "file_size": 1024000,
      "description": "Annual contract 2025",
      "category_unique_id": "cat-123",
      "tags": "[\"legal\", \"2025\"]",
      "is_public": false,
      "ai_enabled": false
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Generated filename (UUID recommended) |
| `original_name` | string | Yes | Original user filename |
| `url` | string | Yes | S3 URL from presigned upload |
| `file_type` | string | Yes | MIME type |
| `file_size` | integer | Yes | Size in bytes |
| `description` | string | No | File description |
| `category_unique_id` | string | No | Category ID |
| `tags` | string | No | JSON array of tags |
| `is_public` | boolean | No | Publish immediately (default: false) |
| `ai_enabled` | boolean | No | Enable AI/RAG processing |

**Response 201:**
```json
{
  "data": {
    "id": "new-file-id",
    "type": "user_file",
    "attributes": {
      "unique_id": "new-file-id",
      "name": "uuid-generated-name.pdf",
      "original_name": "contract.pdf",
      "url": "https://s3.us-east-2.amazonaws.com/...",
      "status": "review",
      "is_public": false,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 uf754321` - Duplicate file (same name, size, type exists)

**Required Scopes:** `user_files:write`

---

### PUT /users/:unique_id/files/:unique_file_id - Update File

Updates file metadata.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "description": "Updated description",
      "category_unique_id": "cat-456",
      "tags": "[\"legal\", \"2025\", \"updated\"]"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "file-id",
    "type": "user_file",
    "attributes": {
      "unique_id": "file-id",
      "description": "Updated description",
      "updated_at": "2025-01-10T14:00:00Z"
    }
  }
}
```

**Required Scopes:** `user_files:write`

---

### DELETE /users/:unique_id/files/:unique_file_id - Delete File

Soft-deletes a file (marks as deleted, removes from S3).

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - File not found
- `403 uf832090` - Insufficient permissions

**Required Scopes:** `user_files:write`

---

### PUT /users/:unique_id/files/:unique_file_id/approve - Approve File

Sets file status to active.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/approve" \
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
      "status": "active"
    }
  }
}
```

**Required Scopes:** `user_files:write`

---

### PUT /users/:unique_id/files/:unique_file_id/reject - Reject File

Sets file status back to review.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/reject" \
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
      "status": "review"
    }
  }
}
```

**Required Scopes:** `user_files:write`

---

### PUT /users/:unique_id/files/:unique_file_id/publish - Publish File

Copies file to public bucket and makes it publicly accessible.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `500 uf832167` - Error publishing file

**Required Scopes:** `user_files:write`

---

### PUT /users/:unique_id/files/:unique_file_id/unpublish - Unpublish File

Removes file from public bucket and makes it private.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/unpublish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `500 uf832168` - Error unpublishing file

**Required Scopes:** `user_files:write`
