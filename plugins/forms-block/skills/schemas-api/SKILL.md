---
name: forms-schemas-api
description: Create and manage 23blocks form schemas via REST API. Use when building backend form functionality, defining schema structures, or debugging API issues.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Forms Schemas API

Complete API reference for 23blocks form schema management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://forms.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |
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

## Endpoints

### POST /schemas - Create Form Schema

Creates a new form schema definition.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/forms/schemas" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "contact_form",
    "description": "Contact us form",
    "fields": [
      {
        "name": "email",
        "type": "email",
        "label": "Email Address",
        "required": true,
        "placeholder": "you@example.com",
        "validation": {
          "format": "email"
        }
      },
      {
        "name": "subject",
        "type": "select",
        "label": "Subject",
        "required": true,
        "options": [
          {"value": "support", "label": "Support"},
          {"value": "sales", "label": "Sales"},
          {"value": "other", "label": "Other"}
        ]
      },
      {
        "name": "message",
        "type": "textarea",
        "label": "Message",
        "required": true,
        "validation": {
          "minLength": 10,
          "maxLength": 2000
        }
      }
    ],
    "settings": {
      "submitLabel": "Send Message",
      "successMessage": "Thanks! We will get back to you soon."
    }
  }'
```

**Response 201:**
```json
{
  "id": "frm_abc123",
  "name": "contact_form",
  "description": "Contact us form",
  "fields": [...],
  "settings": {...},
  "version": 1,
  "status": "active",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

**Errors:**
- `400 INVALID_SCHEMA` - Schema definition is invalid
- `409 NAME_EXISTS` - Schema with this name already exists

---

### GET /schemas/:id - Get Schema

Retrieves a form schema by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/forms/schemas/frm_abc123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN"
```

**Response 200:**
```json
{
  "id": "frm_abc123",
  "name": "contact_form",
  "description": "Contact us form",
  "fields": [
    {
      "name": "email",
      "type": "email",
      "label": "Email Address",
      "required": true,
      "placeholder": "you@example.com",
      "validation": {
        "format": "email"
      }
    }
  ],
  "settings": {
    "submitLabel": "Send Message",
    "successMessage": "Thanks!"
  },
  "version": 1,
  "status": "active",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

**Errors:**
- `404 SCHEMA_NOT_FOUND` - Schema does not exist

---

### PUT /schemas/:id - Update Schema

Updates an existing form schema. Creates a new version.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/forms/schemas/frm_abc123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "description": "Updated contact form",
    "fields": [
      {
        "name": "email",
        "type": "email",
        "label": "Your Email",
        "required": true
      },
      {
        "name": "phone",
        "type": "tel",
        "label": "Phone (optional)",
        "required": false
      },
      {
        "name": "message",
        "type": "textarea",
        "label": "Message",
        "required": true
      }
    ]
  }'
```

**Response 200:**
```json
{
  "id": "frm_abc123",
  "name": "contact_form",
  "version": 2,
  "status": "active",
  "updated_at": "2024-01-16T14:00:00Z"
}
```

**Errors:**
- `404 SCHEMA_NOT_FOUND` - Schema does not exist
- `400 INVALID_SCHEMA` - Updated schema is invalid

---

### DELETE /schemas/:id - Delete Schema

Soft-deletes a form schema. Existing submissions are preserved.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/forms/schemas/frm_abc123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN"
```

**Response 204:** No content

**Errors:**
- `404 SCHEMA_NOT_FOUND` - Schema does not exist

---

### GET /schemas - List Schemas

Lists all form schemas with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/forms/schemas?page=1&limit=20&status=active" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN"
```

**Query Parameters:**
- `page` (integer, default: 1) - Page number
- `limit` (integer, default: 20, max: 100) - Items per page
- `status` (string) - Filter by status: active, archived
- `search` (string) - Search by name or description

**Response 200:**
```json
{
  "data": [
    {
      "id": "frm_abc123",
      "name": "contact_form",
      "description": "Contact us form",
      "status": "active",
      "version": 2,
      "submissions_count": 150,
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "pages": 3
  }
}
```

---

## Data Models

### Schema
| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier (frm_*) |
| `name` | string | URL-safe name, unique |
| `description` | string | Human-readable description |
| `fields` | Field[] | Array of field definitions |
| `settings` | Settings | Form configuration |
| `version` | integer | Schema version number |
| `status` | enum | active, archived |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update time |

### Field
| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Field identifier |
| `type` | enum | text, email, tel, number, select, multiselect, checkbox, textarea, date, datetime, file, image |
| `label` | string | Display label |
| `required` | boolean | Whether field is required |
| `placeholder` | string | Placeholder text |
| `default` | any | Default value |
| `options` | Option[] | For select/multiselect types |
| `validation` | Validation | Validation rules |
| `conditions` | Condition[] | Show/hide conditions |

### Validation
| Field | Type | Description |
|-------|------|-------------|
| `format` | string | email, url, phone, uuid |
| `pattern` | string | Regex pattern |
| `minLength` | integer | Minimum string length |
| `maxLength` | integer | Maximum string length |
| `min` | number | Minimum numeric value |
| `max` | number | Maximum numeric value |
| `unique` | boolean | Must be unique in database |
| `custom` | string | Custom validator name |

---

## Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Schema validation failed",
    "details": {
      "fields[0].type": "Invalid field type: 'unknown'"
    }
  }
}
```
