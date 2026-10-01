---
name: 23blocks-company-positions-api
description: "Company Block positions with levels and departments: CRUD. Use when defining job positions."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.1"
  verified-by: 23blocks-api-company
  verified-date: "2026-05-18"
---

# Positions API

Complete API reference for 23blocks position management within organizational structure.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://company.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /positions - List Positions

Lists all positions with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/positions?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "pos-uuid-123",
      "type": "position",
      "attributes": {
        "unique_id": "pos-uuid-123",
        "title": "Senior Software Engineer",
        "description": "Designs and implements complex software systems",
        "department_id": "dept-uuid-123",
        "level": "senior",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "pos-uuid-456",
      "type": "position",
      "attributes": {
        "unique_id": "pos-uuid-456",
        "title": "Product Manager",
        "description": "Manages product roadmap and priorities",
        "department_id": "dept-uuid-456",
        "level": "mid",
        "status": "active",
        "created_at": "2025-01-11T09:00:00Z",
        "updated_at": "2025-01-11T09:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 4,
    "totalRecords": 56
  },
  "links": {
    "self": "/positions?page=1&records=20",
    "next": "/positions?page=2&records=20",
    "prev": null
  }
}
```

---

### GET /positions/:unique_id - Get Position

Retrieves a single position by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/positions/pos-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "pos-uuid-123",
    "type": "position",
    "attributes": {
      "unique_id": "pos-uuid-123",
      "title": "Senior Software Engineer",
      "description": "Designs and implements complex software systems",
      "department_id": "dept-uuid-123",
      "level": "senior",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "department": {
        "data": { "id": "dept-uuid-123", "type": "department" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Position not found

---

### POST /positions - Create Position

Creates a new position.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/positions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "position": {
      "title": "Staff Engineer",
      "description": "Technical leadership across multiple teams",
      "department_id": "dept-uuid-123",
      "level": "staff",
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | Position title |
| `description` | string | No | Position description |
| `department_id` | uuid | No | Associated department ID |
| `level` | string | No | Position level (e.g., junior, mid, senior, staff, principal) |
| `status` | string | No | active, inactive (default: active) |

**Response 201:**
```json
{
  "data": {
    "id": "new-pos-uuid",
    "type": "position",
    "attributes": {
      "unique_id": "new-pos-uuid",
      "title": "Staff Engineer",
      "description": "Technical leadership across multiple teams",
      "department_id": "dept-uuid-123",
      "level": "staff",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Position title already exists in department
- `422 Unprocessable Entity` - Validation errors

---

### PUT /positions/:unique_id - Update Position

Updates an existing position.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/positions/pos-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "position": {
      "title": "Senior Software Engineer II",
      "description": "Updated role description",
      "level": "senior"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "pos-uuid-123",
    "type": "position",
    "attributes": {
      "unique_id": "pos-uuid-123",
      "title": "Senior Software Engineer II",
      "description": "Updated role description",
      "department_id": "dept-uuid-123",
      "level": "senior",
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Position not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /positions/:unique_id - Delete Position

Deletes a position.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/positions/pos-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** Empty body `{}`

**Errors:**
- `404 Not Found` - Position not found
- `409 Conflict` - Position has active employee assignments

---

## Data Models

### Position
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Position title |
| `description` | string | Position description |
| `department_id` | uuid | Associated department ID |
| `level` | string | Position level (junior, mid, senior, staff, principal) |
| `status` | string | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","source":{"pointer":"/data/attributes/title"},"code":"validation_error","title":"Validation Error","detail":"Title can't be blank."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-company`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCompanyBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// Positions — client.company.positions
client.company.positions.list(params?: ListPositionsParams): Promise<PageResult<Position>>;
client.company.positions.get(uniqueId: string): Promise<Position>;
client.company.positions.create(data: CreatePositionRequest): Promise<Position>;
client.company.positions.update(uniqueId: string, data: UpdatePositionRequest): Promise<Position>;
client.company.positions.delete(uniqueId: string): Promise<void>;
client.company.positions.listByDepartment(departmentUniqueId: string): Promise<Position[]>;
```

### TypeScript Types

```typescript
import type {
  Position,
  CreatePositionRequest,
  UpdatePositionRequest,
  ListPositionsParams,
} from '@23blocks/block-company';
```
