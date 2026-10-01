---
name: 23blocks-files-entity-files-api
description: "Files Block files attached to business entities: upload, associate, share, disassociate. Use when files belong to a record, not a person."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Entity Files API

Complete API reference for 23blocks entity file management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://files.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## File upload: `name` is the presign `file_name`

The Files API generates UUID-based S3 keys during presign. Use the `file_name` returned by the presign endpoint as the `name` field when registering file metadata. Using any other value (such as the original filename) causes **404 errors on download** because the S3 object key will not match.

**Correct single-file upload flow:**
1. `PUT /entities/:id/presign?filename=policy.pdf` returns `{ "file_name": "dcca6ec1-...pdf", "signed_url": "..." }`
2. `PUT {signed_url}` with file bytes
3. `POST /entities/:id/files` with `{ "name": "dcca6ec1-...pdf", "original_name": "policy.pdf" }` -- `name` is `file_name` from step 1

**Correct multipart upload flow (large files):**
1. `POST /entities/:id/multipart_presign_upload` with `{ "filename": "large.pptx", "part_count": 3 }` returns `{ "file_name": "a1b2c3d4-...pptx", "upload_id": "...", "presigned_urls": [...] }`
2. Upload each part to its presigned URL, collect ETags
3. `POST /entities/:id/multipart_complete_upload` with `{ "file_name": "a1b2c3d4-...pptx", "upload_id": "...", "parts": [...] }`
4. `POST /entities/:id/files` with `{ "name": "a1b2c3d4-...pptx", "original_name": "large.pptx" }` -- `name` is `file_name` from step 1

**Common mistake (causes 404 on download):**
```
PUT /presign?filename=policy.pdf  -->  { "file_name": "dcca6ec1-...pdf" }
POST /files with { "name": "policy.pdf" }  <-- wrong: S3 key mismatch, 404 on download
```

**Field meanings:**
| Field | Purpose | Example |
|-------|---------|---------|
| `name` | S3 object key (UUID-based). Used for downloads; not user-facing. | `dcca6ec1-4f3a-4b2e-9a1c-8d7e6f5a4b3c.pdf` |
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Failed","detail":"Entity not found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-files`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useFilesBlock()` from `@23blocks/react`.

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
