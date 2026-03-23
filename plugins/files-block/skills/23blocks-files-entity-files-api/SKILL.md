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
