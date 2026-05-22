---
name: 23blocks-company-departments-api
description: Create and manage 23blocks departments via REST API. Use when creating, updating, or deleting departments, or managing departmental hierarchy with parent-child relationships.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.1"
  verified-by: 23blocks-api-company
  verified-date: "2026-05-18"
---

# Departments API

Complete API reference for 23blocks department management with hierarchical structure.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Company API base URL | `https://company.api.us.23blocks.com` |
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
export BLOCKS_API_URL="https://company.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://company.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### GET /departments - List Departments

Lists all departments with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/departments?page=1&records=20" \
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
      "id": "dept-uuid-123",
      "type": "department",
      "attributes": {
        "unique_id": "dept-uuid-123",
        "name": "Engineering",
        "description": "Software engineering division",
        "parent_department_id": null,
        "manager_id": "user-uuid-001",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "dept-uuid-456",
      "type": "department",
      "attributes": {
        "unique_id": "dept-uuid-456",
        "name": "Frontend Engineering",
        "description": "Frontend development team",
        "parent_department_id": "dept-uuid-123",
        "manager_id": "user-uuid-002",
        "status": "active",
        "created_at": "2025-01-11T09:00:00Z",
        "updated_at": "2025-01-11T09:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 42
  },
  "links": {
    "self": "/departments?page=1&records=20",
    "next": "/departments?page=2&records=20",
    "prev": null
  }
}
```

---

### GET /departments/:unique_id - Get Department

Retrieves a single department by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/departments/dept-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "dept-uuid-123",
    "type": "department",
    "attributes": {
      "unique_id": "dept-uuid-123",
      "name": "Engineering",
      "description": "Software engineering division",
      "parent_department_id": null,
      "manager_id": "user-uuid-001",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "children": {
        "data": [
          { "id": "dept-uuid-456", "type": "department" },
          { "id": "dept-uuid-789", "type": "department" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Department not found

---

### POST /departments - Create Department

Creates a new department.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/departments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "department": {
      "name": "Product",
      "description": "Product management and strategy",
      "parent_department_id": null,
      "manager_id": "user-uuid-003",
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Department name |
| `description` | string | No | Department description |
| `parent_department_id` | uuid | No | Parent department ID (null for root) |
| `manager_id` | uuid | No | Manager user ID |
| `status` | string | No | active, inactive (default: active) |

**Response 201:**
```json
{
  "data": {
    "id": "new-dept-uuid",
    "type": "department",
    "attributes": {
      "unique_id": "new-dept-uuid",
      "name": "Product",
      "description": "Product management and strategy",
      "parent_department_id": null,
      "manager_id": "user-uuid-003",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Department name already exists
- `422 Unprocessable Entity` - Validation errors

---

### PUT /departments/:unique_id - Update Department

Updates an existing department.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/departments/dept-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "department": {
      "name": "Engineering & Platform",
      "description": "Updated description",
      "manager_id": "user-uuid-005"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "dept-uuid-123",
    "type": "department",
    "attributes": {
      "unique_id": "dept-uuid-123",
      "name": "Engineering & Platform",
      "description": "Updated description",
      "parent_department_id": null,
      "manager_id": "user-uuid-005",
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Department not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /departments/:unique_id - Delete Department

Deletes a department.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/departments/dept-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** Empty body `{}`

**Errors:**
- `404 Not Found` - Department not found
- `409 Conflict` - Department has child departments or active assignments

---

## Data Models

### Department
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Department name |
| `description` | string | Department description |
| `parent_department_id` | uuid | Parent department ID (null for root) |
| `manager_id` | uuid | Manager user ID |
| `status` | string | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "source": { "pointer": "/data/attributes/name" },
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Name can't be blank."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-company
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
// Departments — client.company.departments
client.company.departments.list(params?: ListDepartmentsParams): Promise<PageResult<Department>>;
client.company.departments.get(uniqueId: string): Promise<Department>;
client.company.departments.create(data: CreateDepartmentRequest): Promise<Department>;
client.company.departments.update(uniqueId: string, data: UpdateDepartmentRequest): Promise<Department>;
client.company.departments.delete(uniqueId: string): Promise<void>;
client.company.departments.listByCompany(companyUniqueId: string): Promise<Department[]>;
client.company.departments.getHierarchy(companyUniqueId: string): Promise<DepartmentHierarchy[]>;
```

### TypeScript Types

```typescript
import type {
  Department,
  DepartmentHierarchy,
  CreateDepartmentRequest,
  UpdateDepartmentRequest,
  ListDepartmentsParams,
} from '@23blocks/block-company';
```

### React Hook

```typescript
import { useCompanyBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCompanyBlock();
  const result = await client.company.departments.list();
}
```
