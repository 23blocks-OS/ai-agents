---
name: search-entities-api
description: Manage 23blocks search entities via REST API. Use when creating, updating, or deleting entities, listing entity types, retrieving entity type schemas, or managing entities with flexible JSONB content.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Entities API

Complete API reference for 23blocks entity management with flexible JSONB content, types, and schemas.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://search.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Search API base URL | `https://search.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/entities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /entities - List Entities

Lists all entities with pagination, search, and sorting.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities?page=1&records=20&entity_type=product" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `entity_type` | string | No | Filter by entity type |
| `search` | string | No | Text search filter |
| `sort_by` | string | No | Field to sort by |
| `sort_order` | string | No | asc or desc (default: desc) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "entity-uuid-123",
      "type": "entity_identity",
      "attributes": {
        "unique_id": "entity-uuid-123",
        "entity_unique_id": "entity-uuid-123",
        "entity_type": "product",
        "entity_alias": "Widget Pro",
        "entity_source": "api",
        "slug": "widget-pro",
        "content": {
          "name": "Widget Pro",
          "category": "electronics",
          "price": 29.99
        },
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 87
  }
}
```

---

### GET /entities/:unique_id - Get Entity

Retrieves a single entity by unique ID or slug.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/entity-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "entity-uuid-123",
    "type": "entity_identity",
    "attributes": {
      "unique_id": "entity-uuid-123",
      "entity_unique_id": "entity-uuid-123",
      "entity_type": "product",
      "entity_alias": "Widget Pro",
      "entity_source": "api",
      "slug": "widget-pro",
      "content": {
        "name": "Widget Pro",
        "category": "electronics",
        "price": 29.99,
        "description": "Advanced widget with AI features"
      },
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Entity not found

---

### POST /entities/:unique_id/register - Create/Update Entity

Creates a new entity or updates an existing one. The unique_id in the path becomes the entity's identifier.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/entity-uuid-123/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "entity": {
      "entity_type": "product",
      "entity_alias": "Widget Pro",
      "entity_source": "api",
      "content": {
        "name": "Widget Pro",
        "category": "electronics",
        "price": 29.99,
        "description": "Advanced widget with AI features"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `entity_type` | string | Yes | Entity type (e.g., product, article) |
| `entity_alias` | string | No | Human-readable alias/name |
| `entity_source` | string | No | Data source identifier |
| `content` | object | Yes | JSONB content payload |

**Response 201:**
```json
{
  "data": {
    "id": "entity-uuid-123",
    "type": "entity_identity",
    "attributes": {
      "unique_id": "entity-uuid-123",
      "entity_type": "product",
      "entity_alias": "Widget Pro",
      "slug": "widget-pro",
      "content": {
        "name": "Widget Pro",
        "category": "electronics",
        "price": 29.99
      },
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors or schema violation

---

### PUT /entities/:unique_id - Update Entity

Updates an existing entity.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/entities/entity-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "entity": {
      "entity_alias": "Widget Pro v2",
      "content": {
        "name": "Widget Pro v2",
        "category": "electronics",
        "price": 34.99
      }
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "entity-uuid-123",
    "type": "entity_identity",
    "attributes": {
      "unique_id": "entity-uuid-123",
      "entity_alias": "Widget Pro v2",
      "content": {
        "name": "Widget Pro v2",
        "category": "electronics",
        "price": 34.99
      },
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /entities/:unique_id - Delete Entity

Deletes an entity.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/entities/entity-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Entity not found

---

### GET /entity_types - List Entity Types

Lists all registered entity types.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entity_types" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "type-uuid-1",
      "type": "entity_type",
      "attributes": {
        "entity_type": "product",
        "entity_source": "api"
      }
    },
    {
      "id": "type-uuid-2",
      "type": "entity_type",
      "attributes": {
        "entity_type": "article",
        "entity_source": "cms"
      }
    }
  ]
}
```

---

### GET /entity_types/:entity_type - Get Entity Type Schema

Retrieves the JSON schema definition for a specific entity type. Used for validation of entity content.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entity_types/product" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "schema-uuid-123",
    "type": "entity_type_schema",
    "attributes": {
      "entity_type": "product",
      "enforce_validation": true,
      "schema": {
        "type": "object",
        "required": ["name", "category"],
        "properties": {
          "name": { "type": "string" },
          "category": { "type": "string" },
          "price": { "type": "number" },
          "description": { "type": "string" }
        }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Entity type not found

---

## Data Models

### EntityIdentity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `entity_unique_id` | uuid | Entity unique ID |
| `entity_type` | string | Type classification |
| `entity_alias` | string | Human-readable name |
| `entity_source` | string | Data source identifier |
| `slug` | string | URL-friendly slug |
| `content` | object | JSONB content payload |
| `ai_vector` | vector | AI embedding vector (pgvector) |
| `status` | string | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### EntityType
| Field | Type | Description |
|-------|------|-------------|
| `entity_type` | string | Type name |
| `entity_source` | string | Source system |

### EntityTypeSchema
| Field | Type | Description |
|-------|------|-------------|
| `entity_type` | string | Type this schema applies to |
| `schema` | object | JSON Schema definition |
| `enforce_validation` | boolean | Whether to enforce on create/update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "schema_validation_error",
    "title": "Content Validation Failed",
    "detail": "Entity content does not match the schema for type 'product'."
  }]
}
```
