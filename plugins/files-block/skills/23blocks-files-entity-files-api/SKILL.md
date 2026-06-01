---
name: 23blocks-files-entity-files-api
description: Manage entity files via REST API. Use when associating files with business entities, sharing files between entities, or managing entity-level file operations.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Entity Files API

Complete API reference for 23blocks entity file management.

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

**Correct single-file upload flow:**
1. `PUT /entities/:id/presign?filename=policy.pdf` returns `{ "file_name": "dcca6ec1-...pdf", "signed_url": "..." }`
2. `PUT {signed_url}` with file bytes
3. `POST /entities/:id/files` with `{ "name": "dcca6ec1-...pdf", "original_name": "policy.pdf" }` -- `name` **MUST** match `file_name` from step 1

**Correct multipart upload flow (large files):**
1. `POST /entities/:id/multipart_presign_upload` with `{ "filename": "large.pptx", "part_count": 3 }` returns `{ "file_name": "a1b2c3d4-...pptx", "upload_id": "...", "presigned_urls": [...] }`
2. Upload each part to its presigned URL, collect ETags
3. `POST /entities/:id/multipart_complete_upload` with `{ "file_name": "a1b2c3d4-...pptx", "upload_id": "...", "parts": [...] }`
4. `POST /entities/:id/files` with `{ "name": "a1b2c3d4-...pptx", "original_name": "large.pptx" }` -- `name` **MUST** match `file_name` from step 1

**Common mistake (causes 404 on download):**
```
PUT /presign?filename=policy.pdf  -->  { "file_name": "dcca6ec1-...pdf" }
POST /files with { "name": "policy.pdf" }  <-- WRONG! S3 key mismatch = 404
```

**Field meanings:**
| Field | Purpose | Example |
|-------|---------|---------|
| `name` | S3 object key (UUID-based). Used for downloads. NOT user-facing. | `dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.pdf` |
| `original_name` | User's original filename. Used for display only. | `policy.pdf` |

---

## Overview

Entity files are associated with business entities (companies, organizations, projects, etc.) rather than individual users. This is useful for:
- Company documents
- Project files
- Shared team resources
- Entity-level attachments

---

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/entities` | List entities |
| GET | `/entities/:unique_id` | Get entity |
| POST | `/entities/:unique_id/register` | Register entity |
| GET | `/entities/:unique_id/files` | List entity files |
| GET | `/entities/:unique_id/files/:unique_file_id` | Get entity file |
| PUT | `/entities/:unique_id/presign` | Get presigned URL |
| POST | `/entities/:unique_id/multipart_presign_upload` | Start multipart upload |
| POST | `/entities/:unique_id/multipart_complete_upload` | Complete multipart upload |
| POST | `/entities/:unique_id/files` | Create entity file |
| PUT | `/entities/:unique_id/files/:unique_file_id` | Update entity file |
| DELETE | `/entities/:unique_id/files/:unique_file_id` | Delete entity file |
| POST | `/entities/:unique_id/files/associate` | Associate file |
| DELETE | `/entities/:unique_id/files/:unique_file_id/disassociate` | Disassociate file |

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
| `name` | string | UUID-based S3 key (from presign `file_name`). Used for downloads. |
| `original_name` | string | User's original filename (display only) |
| `url` | string | S3 URL |
| `thumbnail_url` | string | Thumbnail URL |
| `file_type` | string | MIME type |
| `file_size` | integer | Size in bytes |
| `description` | string | Description |
| `status` | enum | active, deleted |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Failed",
    "detail": "Entity not found."
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
// EntityFilesService — client.files.entityFiles
list(params?: ListEntityFilesParams): Promise<PageResult<EntityFile>>;
get(uniqueId: string): Promise<EntityFile>;
attach(data: AttachFileRequest): Promise<EntityFile>;
detach(uniqueId: string): Promise<void>;
update(uniqueId: string, data: UpdateEntityFileRequest): Promise<EntityFile>;
reorder(entityUniqueId: string, entityType: string, data: ReorderFilesRequest): Promise<EntityFile[]>;
listByEntity(entityUniqueId: string, entityType: string, params?: ListEntityFilesParams): Promise<PageResult<EntityFile>>;
```

### TypeScript Types

```typescript
import type {
  EntityFile,
  AttachFileRequest,
  UpdateEntityFileRequest,
  ListEntityFilesParams,
  ReorderFilesRequest,
} from '@23blocks/block-files';
```

### React Hook

```typescript
import { useFilesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFilesBlock();
  const result = await client.files.entityFiles.list();
}
```
