---
name: crm-leads-api
description: Manage 23blocks CRM leads via REST API. Use when creating, updating, or tracking leads through the sales pipeline, and managing lead follow-up tasks.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# CRM Leads API

Complete API reference for 23blocks CRM lead management with follow-up task tracking.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://crm.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
echo "All credentials configured"
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | CRM API base URL | `https://crm.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/leads/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /leads/ - List Leads

Lists all leads with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/leads/?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
      "id": "lead-uuid-123",
      "type": "lead",
      "attributes": {
        "unique_id": "lead-uuid-123",
        "name": "Enterprise Deal",
        "source": "website",
        "account_id": "account-uuid-456",
        "contact_id": "contact-uuid-789",
        "value": 50000,
        "stage": "qualified",
        "probability": 40,
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 38
  }
}
```

---

### GET /leads/:unique_id - Get Lead

Retrieves a single lead by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/leads/lead-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "lead-uuid-123",
    "type": "lead",
    "attributes": {
      "unique_id": "lead-uuid-123",
      "name": "Enterprise Deal",
      "source": "website",
      "account_id": "account-uuid-456",
      "contact_id": "contact-uuid-789",
      "value": 50000,
      "stage": "qualified",
      "probability": 40,
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    },
    "relationships": {
      "account": {
        "data": { "id": "account-uuid-456", "type": "account" }
      },
      "contact": {
        "data": { "id": "contact-uuid-789", "type": "contact" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Lead not found

---

### POST /leads/ - Create Lead

Creates a new lead.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/leads/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "lead": {
      "name": "New Enterprise Deal",
      "source": "referral",
      "account_id": "account-uuid-456",
      "contact_id": "contact-uuid-789",
      "value": 75000,
      "stage": "new",
      "probability": 20
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Lead name |
| `source` | string | No | Lead source (website, referral, cold_call, etc.) |
| `account_id` | uuid | No | Associated account ID |
| `contact_id` | uuid | No | Associated contact ID |
| `value` | decimal | No | Estimated deal value |
| `stage` | string | No | Pipeline stage (new, contacted, qualified, proposal, negotiation) |
| `probability` | integer | No | Win probability percentage (0-100) |

**Response 201:**
```json
{
  "data": {
    "id": "lead-uuid-456",
    "type": "lead",
    "attributes": {
      "unique_id": "lead-uuid-456",
      "name": "New Enterprise Deal",
      "source": "referral",
      "account_id": "account-uuid-456",
      "contact_id": "contact-uuid-789",
      "value": 75000,
      "stage": "new",
      "probability": 20,
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /leads/:unique_id/ - Update Lead

Updates an existing lead.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/leads/lead-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "lead": {
      "stage": "proposal",
      "probability": 60,
      "value": 65000
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "lead-uuid-123",
    "type": "lead",
    "attributes": {
      "unique_id": "lead-uuid-123",
      "stage": "proposal",
      "probability": 60,
      "value": 65000,
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Lead not found

---

### DELETE /leads/:unique_id/ - Delete Lead

Deletes a lead.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/leads/lead-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Lead not found

---

## Lead Follows

Follow-up tasks associated with leads for tracking next actions.

### GET /leads/:unique_id/follows - List Follows

Lists all follow-up tasks for a lead.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/leads/lead-uuid-123/follows" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "follow-uuid-123",
      "type": "follow",
      "attributes": {
        "unique_id": "follow-uuid-123",
        "title": "Send proposal document",
        "description": "Prepare and send the Q2 proposal",
        "due_date": "2026-02-28T17:00:00Z",
        "status": "pending",
        "created_at": "2026-01-15T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /leads/:unique_id/follows/:follow_unique_id - Get Follow

Retrieves a specific follow-up task.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/leads/lead-uuid-123/follows/follow-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "follow-uuid-123",
    "type": "follow",
    "attributes": {
      "unique_id": "follow-uuid-123",
      "title": "Send proposal document",
      "description": "Prepare and send the Q2 proposal",
      "due_date": "2026-02-28T17:00:00Z",
      "status": "pending",
      "created_at": "2026-01-15T10:30:00Z"
    }
  }
}
```

---

### POST /leads/:unique_id/follows - Create Follow

Creates a new follow-up task for a lead.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/leads/lead-uuid-123/follows" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "follow": {
      "title": "Schedule demo call",
      "description": "Set up product demo with technical team",
      "due_date": "2026-03-05T14:00:00Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | Follow-up task title |
| `description` | string | No | Task description |
| `due_date` | timestamp | No | Due date |

**Response 201:**
```json
{
  "data": {
    "id": "follow-uuid-456",
    "type": "follow",
    "attributes": {
      "unique_id": "follow-uuid-456",
      "title": "Schedule demo call",
      "description": "Set up product demo with technical team",
      "due_date": "2026-03-05T14:00:00Z",
      "status": "pending",
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

---

### PUT /leads/:unique_id/follows/:follow_unique_id - Update Follow

Updates an existing follow-up task.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/leads/lead-uuid-123/follows/follow-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "follow": {
      "status": "completed",
      "description": "Proposal sent and received"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "follow-uuid-123",
    "type": "follow",
    "attributes": {
      "unique_id": "follow-uuid-123",
      "status": "completed",
      "description": "Proposal sent and received",
      "updated_at": "2026-02-15T14:00:00Z"
    }
  }
}
```

---

### DELETE /leads/:unique_id/follows/:follow_unique_id - Delete Follow

Deletes a follow-up task.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/leads/lead-uuid-123/follows/follow-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Lead
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Lead name |
| `source` | string | Lead source (website, referral, cold_call, etc.) |
| `account_id` | uuid | Associated account ID |
| `contact_id` | uuid | Associated contact ID |
| `value` | decimal | Estimated deal value |
| `stage` | string | Pipeline stage |
| `probability` | integer | Win probability (0-100) |
| `status` | enum | active, inactive, converted, lost |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Follow
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Task title |
| `description` | string | Task description |
| `due_date` | timestamp | Due date |
| `status` | enum | pending, completed, overdue |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Lead Not Found",
    "detail": "The requested lead could not be found."
  }]
}
```
