# Contexts API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Endpoints

### List Contexts

Retrieve all contexts.

**Endpoint:** `GET /contexts/`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number for pagination |
| per_page | integer | No | Results per page (default: 25) |
| context_type | string | No | Filter by context type |
| status | string | No | Filter by status: `active`, `archived` |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/contexts/?page=1&per_page=25&status=active" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "ctx_proj_alpha",
      "type": "contexts",
      "attributes": {
        "unique_id": "ctx_proj_alpha",
        "name": "Project Alpha",
        "description": "All conversations related to Project Alpha",
        "context_type": "project",
        "metadata": {
          "project_code": "ALPHA-001",
          "department": "engineering"
        },
        "status": "active",
        "group_count": 3,
        "created_at": "2025-01-05T08:00:00Z",
        "updated_at": "2025-01-14T16:00:00Z"
      }
    }
  ],
  "meta": {
    "total": 8,
    "page": 1,
    "per_page": 25
  }
}
```

---

### Get Context

Retrieve details of a specific context.

**Endpoint:** `GET /contexts/:unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The context's unique identifier |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/contexts/ctx_proj_alpha" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "ctx_proj_alpha",
    "type": "contexts",
    "attributes": {
      "unique_id": "ctx_proj_alpha",
      "name": "Project Alpha",
      "description": "All conversations related to Project Alpha",
      "context_type": "project",
      "metadata": {
        "project_code": "ALPHA-001",
        "department": "engineering",
        "start_date": "2025-01-01",
        "end_date": "2025-06-30"
      },
      "status": "active",
      "group_count": 3,
      "groups": [
        {
          "unique_id": "grp_ctx001",
          "name": "Project Alpha Team"
        },
        {
          "unique_id": "grp_ctx002",
          "name": "Project Alpha - Backend"
        },
        {
          "unique_id": "grp_ctx003",
          "name": "Project Alpha - Frontend"
        }
      ],
      "created_at": "2025-01-05T08:00:00Z",
      "updated_at": "2025-01-14T16:00:00Z"
    }
  }
}
```

---

### Create Context

Create a new context for organizing groups and conversations.

**Endpoint:** `POST /contexts/`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Context name |
| description | string | No | Context description |
| context_type | string | No | Type: `project`, `department`, `topic`, `custom` |
| metadata | object | No | Additional metadata |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/contexts/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Project Beta",
    "description": "Conversations for the new Project Beta initiative",
    "context_type": "project",
    "metadata": {
      "project_code": "BETA-001",
      "department": "product",
      "start_date": "2025-02-01"
    }
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "ctx_proj_beta",
    "type": "contexts",
    "attributes": {
      "unique_id": "ctx_proj_beta",
      "name": "Project Beta",
      "description": "Conversations for the new Project Beta initiative",
      "context_type": "project",
      "metadata": {
        "project_code": "BETA-001",
        "department": "product",
        "start_date": "2025-02-01"
      },
      "status": "active",
      "group_count": 0,
      "created_at": "2025-01-15T11:00:00Z",
      "updated_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

### Update Context

Update an existing context.

**Endpoint:** `PUT /contexts/:unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The context's unique identifier |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | No | Updated name |
| description | string | No | Updated description |
| context_type | string | No | Updated type |
| metadata | object | No | Updated metadata |
| status | string | No | Updated status: `active`, `archived` |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/contexts/ctx_proj_alpha" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "description": "Updated: All conversations related to Project Alpha - Phase 2",
    "metadata": {
      "project_code": "ALPHA-001",
      "department": "engineering",
      "start_date": "2025-01-01",
      "end_date": "2025-09-30",
      "phase": 2
    }
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "ctx_proj_alpha",
    "type": "contexts",
    "attributes": {
      "unique_id": "ctx_proj_alpha",
      "name": "Project Alpha",
      "description": "Updated: All conversations related to Project Alpha - Phase 2",
      "context_type": "project",
      "status": "active",
      "updated_at": "2025-01-15T12:00:00Z"
    }
  }
}
```

---

### List Context Groups

Retrieve all groups associated with a specific context.

**Endpoint:** `GET /context/:context_unique_id/groups`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| context_unique_id | string | Yes | The context's unique identifier |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number for pagination |
| per_page | integer | No | Results per page (default: 25) |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/context/ctx_proj_alpha/groups?page=1&per_page=25" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "grp_ctx001",
      "type": "groups",
      "attributes": {
        "unique_id": "grp_ctx001",
        "name": "Project Alpha Team",
        "description": "Core project team",
        "context_id": "ctx_proj_alpha",
        "member_count": 8,
        "status": "active",
        "created_at": "2025-01-05T09:00:00Z"
      }
    }
  ],
  "meta": {
    "total": 3,
    "page": 1,
    "per_page": 25
  }
}
```
