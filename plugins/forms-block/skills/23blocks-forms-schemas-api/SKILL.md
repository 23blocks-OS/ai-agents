---
name: 23blocks-forms-schemas-api
description: "Forms Block schemas over REST: a form's field structure. Use when defining or changing fields server-side."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Forms Schemas API

Complete API reference for 23blocks form schema management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://forms.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### POST /schemas - Create Form Schema

Creates a new form schema definition.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/forms/schemas" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "form_schema": {
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
    "form_schema": {
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
    }
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

---

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-forms`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// FormSchemasService — client.forms.schemas
list(formUniqueId: string, params?: ListFormSchemasParams): Promise<PageResult<FormSchema>>;
get(formUniqueId: string, schemaUniqueId: string): Promise<FormSchema>;
create(formUniqueId: string, data: CreateFormSchemaRequest): Promise<FormSchema>;
update(formUniqueId: string, schemaUniqueId: string, data: UpdateFormSchemaRequest): Promise<FormSchema>;
delete(formUniqueId: string, schemaUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  FormSchema,
  CreateFormSchemaRequest,
  UpdateFormSchemaRequest,
  ListFormSchemasParams,
} from '@23blocks/block-forms';
```

### React Hook

```typescript
import { useFormsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFormsBlock();

  // Example: list schemas for a form
  const result = await client.forms.schemas.list('form-unique-id');
}
```
