---
name: files-schemas-api
description: Create and manage file schemas via REST API. Define structured content schemas for files to enable consistent metadata and AI/RAG processing.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# File Schemas API

Complete API reference for 23blocks file schema management.

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
curl -X GET "$BLOCKS_API_URL/schemas" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
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
