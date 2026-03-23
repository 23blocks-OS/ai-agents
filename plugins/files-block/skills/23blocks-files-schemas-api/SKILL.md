---
name: 23blocks-files-schemas-api
description: Create and manage file schemas via REST API. Use when defining structured content schemas for files or configuring AI/RAG processing metadata.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# File Schemas API

Complete API reference for 23blocks file schema management.

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

File schemas define the structure of metadata and content that can be associated with files. They are used for:
- Structured document content (parsed documents)
- Custom metadata fields
- AI/RAG processing schemas
- Content extraction templates

---

## Endpoints

### GET /schemas - List Schemas

Lists all file schemas for the tenant.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/schemas" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "schema-123",
      "type": "file_schema",
      "attributes": {
        "unique_id": "schema-123",
        "name": "Invoice Schema",
        "code": "INVOICE",
        "description": "Schema for invoice documents",
        "schema_definition": {
          "type": "object",
          "properties": {
            "invoice_number": {"type": "string"},
            "amount": {"type": "number"},
            "date": {"type": "string", "format": "date"},
            "vendor": {"type": "string"}
          },
          "required": ["invoice_number", "amount"]
        },
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /schemas/:unique_id - Get Schema

Retrieves a single schema by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/schemas/$SCHEMA_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "schema-123",
    "type": "file_schema",
    "attributes": {
      "unique_id": "schema-123",
      "name": "Invoice Schema",
      "code": "INVOICE",
      "description": "Schema for invoice documents",
      "schema_definition": {
        "type": "object",
        "properties": {
          "invoice_number": {"type": "string"},
          "amount": {"type": "number"},
          "date": {"type": "string", "format": "date"},
          "vendor": {"type": "string"},
          "line_items": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "description": {"type": "string"},
                "quantity": {"type": "number"},
                "unit_price": {"type": "number"}
              }
            }
          }
        },
        "required": ["invoice_number", "amount"]
      },
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Schema not found

---

### POST /schemas - Create Schema

Creates a new file schema.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/schemas" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "schema": {
      "name": "Contract Schema",
      "code": "CONTRACT",
      "description": "Schema for legal contracts",
      "schema_definition": {
        "type": "object",
        "properties": {
          "contract_id": {"type": "string"},
          "parties": {
            "type": "array",
            "items": {"type": "string"}
          },
          "effective_date": {"type": "string", "format": "date"},
          "expiration_date": {"type": "string", "format": "date"},
          "value": {"type": "number"},
          "terms": {"type": "string"}
        },
        "required": ["contract_id", "parties", "effective_date"]
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Schema display name |
| `code` | string | Yes | Unique schema code (uppercase) |
| `description` | string | No | Schema description |
| `schema_definition` | object | Yes | JSON Schema definition |

**Response 201:**
```json
{
  "data": {
    "id": "new-schema-id",
    "type": "file_schema",
    "attributes": {
      "unique_id": "new-schema-id",
      "name": "Contract Schema",
      "code": "CONTRACT",
      "description": "Schema for legal contracts",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### PUT /schemas/:unique_id - Update Schema

Updates an existing schema.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/schemas/$SCHEMA_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "schema": {
      "description": "Updated schema for legal contracts",
      "schema_definition": {
        "type": "object",
        "properties": {
          "contract_id": {"type": "string"},
          "parties": {"type": "array", "items": {"type": "string"}},
          "effective_date": {"type": "string", "format": "date"},
          "expiration_date": {"type": "string", "format": "date"},
          "value": {"type": "number"},
          "currency": {"type": "string"},
          "terms": {"type": "string"}
        },
        "required": ["contract_id", "parties", "effective_date"]
      }
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "schema-id",
    "type": "file_schema",
    "attributes": {
      "description": "Updated schema for legal contracts",
      "updated_at": "2025-01-10T14:00:00Z"
    }
  }
}
```

---

### DELETE /schemas/:unique_id - Delete Schema

Deletes a file schema.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/schemas/$SCHEMA_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### FileSchema
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Display name |
| `code` | string | Unique code (uppercase) |
| `description` | string | Description |
| `schema_definition` | json | JSON Schema definition |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Using Schemas with Files

### Associating a Schema with a File

When creating or updating a file, specify the schema:

```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "name": "uuid-invoice.pdf",
      "original_name": "invoice-2025-001.pdf",
      "file_type": "application/pdf",
      "file_size": 150000,
      "url": "https://s3.us-east-2.amazonaws.com/...",
      "schema_model": "INVOICE",
      "structured_content": "{\"invoice_number\": \"INV-2025-001\", \"amount\": 5000.00, \"date\": \"2025-01-15\", \"vendor\": \"ACME Corp\"}"
    }
  }'
```

### File Fields for Structured Content

| Field | Type | Description |
|-------|------|-------------|
| `schema_model` | string | Schema code to apply |
| `structured_content` | json | Content validated against schema |
| `file_structure` | json | Parsed document structure |
| `metadata` | json | Additional metadata |
| `raw_content` | text | Raw extracted text |
| `content` | text | Processed content |

---

## AI/RAG Integration

Schemas enable AI-powered file processing:

1. **Content Extraction**: Parse documents and extract structured data
2. **Schema Validation**: Validate extracted content against schema
3. **RAG Indexing**: Index structured content for retrieval
4. **Query Processing**: Query files based on schema fields

### Example: Query Files by Schema Field

```bash
# AI query against file content
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/query" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": {
      "unique_id": "prompt-id"
    },
    "message": {
      "content": "What is the total amount of this invoice?"
    }
  }'
```

---

## Common Schema Examples

### Invoice Schema
```json
{
  "type": "object",
  "properties": {
    "invoice_number": {"type": "string"},
    "amount": {"type": "number"},
    "currency": {"type": "string", "default": "USD"},
    "date": {"type": "string", "format": "date"},
    "due_date": {"type": "string", "format": "date"},
    "vendor": {"type": "string"},
    "line_items": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "description": {"type": "string"},
          "quantity": {"type": "number"},
          "unit_price": {"type": "number"}
        }
      }
    }
  },
  "required": ["invoice_number", "amount"]
}
```

### Resume Schema
```json
{
  "type": "object",
  "properties": {
    "name": {"type": "string"},
    "email": {"type": "string", "format": "email"},
    "phone": {"type": "string"},
    "summary": {"type": "string"},
    "experience": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "company": {"type": "string"},
          "title": {"type": "string"},
          "start_date": {"type": "string"},
          "end_date": {"type": "string"},
          "description": {"type": "string"}
        }
      }
    },
    "education": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "institution": {"type": "string"},
          "degree": {"type": "string"},
          "year": {"type": "integer"}
        }
      }
    },
    "skills": {"type": "array", "items": {"type": "string"}}
  },
  "required": ["name"]
}
```

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
// FileSchemasService — client.files.fileSchemas
list(params?: ListFileSchemasParams): Promise<PageResult<FileSchema>>;
get(uniqueId: string): Promise<FileSchema>;
getByCode(code: string): Promise<FileSchema>;
create(data: CreateFileSchemaRequest): Promise<FileSchema>;
update(uniqueId: string, data: UpdateFileSchemaRequest): Promise<FileSchema>;
delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  FileSchema,
  CreateFileSchemaRequest,
  UpdateFileSchemaRequest,
  ListFileSchemasParams,
} from '@23blocks/block-files';
```

### React Hook

```typescript
import { useFilesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFilesBlock();
  const result = await client.files.fileSchemas.list();
}
```
