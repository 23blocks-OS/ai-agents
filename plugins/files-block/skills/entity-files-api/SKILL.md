---
name: files-entity-files-api
description: Manage entity files via REST API. Associate files with business entities, support file sharing between entities, and handle entity-level file operations.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Entity Files API

Complete API reference for 23blocks entity file management.

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
curl -X GET "$BLOCKS_API_URL/entities/{entity_id}/files" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Overview

Entity files are associated with business entities (companies, organizations, projects, etc.) rather than individual users. This is useful for:
- Company documents
- Project files
- Shared team resources
- Entity-level attachments

---

## Entity Management Endpoints

### GET /entities - List Entities

Lists all registered entities.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "entity-123",
      "type": "entity_identity",
      "attributes": {
        "unique_id": "entity-123",
        "entity_alias": "acme-corp",
        "entity_type": "company",
        "name": "ACME Corporation",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /entities/:unique_id - Get Entity

Retrieves a single entity.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/$ENTITY_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "entity-123",
    "type": "entity_identity",
    "attributes": {
      "unique_id": "entity-123",
      "entity_alias": "acme-corp",
      "entity_type": "company",
      "name": "ACME Corporation",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### POST /entities/:unique_id/register - Register Entity

Registers a new entity for file management.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/$ENTITY_ID/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "entity": {
      "entity_alias": "acme-corp",
      "entity_type": "company",
      "name": "ACME Corporation"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `entity_alias` | string | No | Friendly alias for the entity |
| `entity_type` | string | No | Type of entity |
| `name` | string | No | Entity name |

**Response 201:**
```json
{
  "data": {
    "id": "entity-123",
    "type": "entity_identity",
    "attributes": {
      "unique_id": "entity-123",
      "entity_alias": "acme-corp",
      "entity_type": "company",
      "name": "ACME Corporation",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

## Entity Files Endpoints

### GET /entities/:unique_id/files - List Entity Files

Lists all files for an entity.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/$ENTITY_ID/files?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `search` | string | No | Search by filename |

**Response 200:**
```json
{
  "data": [
    {
      "id": "entity-file-123",
      "type": "entity_file",
      "attributes": {
        "unique_id": "entity-file-123",
        "name": "company-policy.pdf",
        "original_name": "policy.pdf",
        "url": "https://s3.us-east-2.amazonaws.com/...",
        "file_type": "application/pdf",
        "file_size": 500000,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 45
  }
}
```

---

### GET /entities/:unique_id/files/:unique_file_id - Get Entity File

Retrieves a single entity file.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/$ENTITY_ID/files/$FILE_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "entity-file-123",
    "type": "entity_file",
    "attributes": {
      "unique_id": "entity-file-123",
      "entity_unique_id": "entity-123",
      "name": "company-policy.pdf",
      "original_name": "policy.pdf",
      "url": "https://s3.us-east-2.amazonaws.com/...?X-Amz-Signature=...",
      "thumbnail_url": "https://s3.us-east-2.amazonaws.com/...",
      "file_type": "application/pdf",
      "file_size": 500000,
      "description": "Company policy document",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### PUT /entities/:unique_id/presign - Get Presigned URL

Gets a presigned URL for uploading a file to an entity.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/entities/$ENTITY_ID/presign?filename=policy.pdf" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "signed_url": "https://s3.us-east-2.amazonaws.com/...?X-Amz-Signature=...",
  "public_url": "https://s3.us-east-2.amazonaws.com/.../policy.pdf"
}
```

---

### POST /entities/:unique_id/multipart_presign_upload - Start Multipart Upload

Initiates a multipart upload for large entity files.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/$ENTITY_ID/multipart_presign_upload" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "large-presentation.pptx",
    "part_count": 3
  }'
```

**Response 200:**
```json
{
  "upload_id": "abc123xyz",
  "presigned_urls": [
    "https://s3...?partNumber=1&uploadId=abc123xyz&X-Amz-Signature=...",
    "https://s3...?partNumber=2&uploadId=abc123xyz&X-Amz-Signature=...",
    "https://s3...?partNumber=3&uploadId=abc123xyz&X-Amz-Signature=..."
  ]
}
```

---

### POST /entities/:unique_id/multipart_complete_upload - Complete Multipart Upload

Finalizes a multipart upload for entity files.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/$ENTITY_ID/multipart_complete_upload" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "large-presentation.pptx",
    "upload_id": "abc123xyz",
    "parts": [
      {"PartNumber": 1, "ETag": "\"abc123\""},
      {"PartNumber": 2, "ETag": "\"def456\""},
      {"PartNumber": 3, "ETag": "\"ghi789\""}
    ]
  }'
```

**Response 200:**
```json
{
  "public_url": "https://s3.us-east-2.amazonaws.com/.../large-presentation.pptx",
  "file_name": "large-presentation.pptx"
}
```

---

### POST /entities/:unique_id/files - Create Entity File

Registers an uploaded file for an entity.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/$ENTITY_ID/files" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "name": "uuid-policy.pdf",
      "original_name": "policy.pdf",
      "url": "https://s3.us-east-2.amazonaws.com/.../uuid-policy.pdf",
      "file_type": "application/pdf",
      "file_size": 500000,
      "description": "Company policy document"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "new-entity-file-id",
    "type": "entity_file",
    "attributes": {
      "unique_id": "new-entity-file-id",
      "name": "uuid-policy.pdf",
      "original_name": "policy.pdf",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### PUT /entities/:unique_id/files/:unique_file_id - Update Entity File

Updates entity file metadata.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/entities/$ENTITY_ID/files/$FILE_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "description": "Updated company policy 2025"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "entity-file-id",
    "type": "entity_file",
    "attributes": {
      "description": "Updated company policy 2025",
      "updated_at": "2025-01-10T14:00:00Z"
    }
  }
}
```

---

### DELETE /entities/:unique_id/files/:unique_file_id - Delete Entity File

Deletes an entity file.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/entities/$ENTITY_ID/files/$FILE_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## File Association Endpoints

### POST /entities/:unique_id/files/associate - Associate File

Associates an existing file with an entity (file sharing between entities).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/$ENTITY_ID/files/associate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file_unique_id": "existing-file-id",
    "source_entity_id": "source-entity-id"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file_unique_id` | string | Yes | File to associate |
| `source_entity_id` | string | No | Source entity (for cross-entity sharing) |

**Response 200:**
```json
{
  "data": {
    "id": "association-id",
    "type": "entity_file",
    "attributes": {
      "file_unique_id": "existing-file-id",
      "entity_unique_id": "entity-id"
    }
  }
}
```

---

### DELETE /entities/:unique_id/files/:unique_file_id/disassociate - Disassociate File

Removes a file association from an entity.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/entities/$ENTITY_ID/files/$FILE_ID/disassociate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### EntityIdentity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `entity_alias` | string | Friendly alias |
| `entity_type` | string | Type of entity |
| `name` | string | Entity name |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### EntityFile
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `entity_unique_id` | uuid | Parent entity ID |
| `name` | string | Generated filename |
| `original_name` | string | Original filename |
| `url` | string | S3 URL |
| `thumbnail_url` | string | Thumbnail URL |
| `file_type` | string | MIME type |
| `file_size` | integer | Size in bytes |
| `description` | string | Description |
| `status` | enum | active, deleted |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |
