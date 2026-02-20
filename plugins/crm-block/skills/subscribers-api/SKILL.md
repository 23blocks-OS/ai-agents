---
name: crm-subscribers-api
description: Manage 23blocks CRM subscribers via REST API. Use when creating, updating, or managing subscriber lists and handling communication unsubscribe requests.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# CRM Subscribers API

Complete API reference for 23blocks CRM subscriber management and communication preferences.

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
curl -X GET "$BLOCKS_API_URL/subscribers/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /subscribers/ - List Subscribers

Lists all subscribers with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subscribers/?page=1&records=20" \
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
      "id": "subscriber-uuid-123",
      "type": "subscriber",
      "attributes": {
        "unique_id": "subscriber-uuid-123",
        "email": "subscriber@example.com",
        "name": "Alice Johnson",
        "source": "website_form",
        "subscribed_at": "2026-01-05T08:00:00Z",
        "status": "active",
        "created_at": "2026-01-05T08:00:00Z",
        "updated_at": "2026-01-05T08:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 10,
    "totalRecords": 142
  }
}
```

---

### GET /subscribers/:unique_id - Get Subscriber

Retrieves a single subscriber by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subscribers/subscriber-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "subscriber-uuid-123",
    "type": "subscriber",
    "attributes": {
      "unique_id": "subscriber-uuid-123",
      "email": "subscriber@example.com",
      "name": "Alice Johnson",
      "source": "website_form",
      "subscribed_at": "2026-01-05T08:00:00Z",
      "status": "active",
      "created_at": "2026-01-05T08:00:00Z",
      "updated_at": "2026-01-05T08:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Subscriber not found

---

### POST /subscribers/ - Create Subscriber

Creates a new subscriber.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/subscribers/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscriber": {
      "email": "newsubscriber@example.com",
      "name": "Bob Williams",
      "source": "landing_page"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Subscriber email address |
| `name` | string | No | Subscriber name |
| `source` | string | No | Subscription source (website_form, landing_page, import, api) |

**Response 201:**
```json
{
  "data": {
    "id": "subscriber-uuid-456",
    "type": "subscriber",
    "attributes": {
      "unique_id": "subscriber-uuid-456",
      "email": "newsubscriber@example.com",
      "name": "Bob Williams",
      "source": "landing_page",
      "subscribed_at": "2026-02-15T10:30:00Z",
      "status": "active",
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (e.g., duplicate email)

---

### PUT /subscribers/:unique_id/ - Update Subscriber

Updates an existing subscriber.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/subscribers/subscriber-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscriber": {
      "name": "Alice M. Johnson",
      "status": "active"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "subscriber-uuid-123",
    "type": "subscriber",
    "attributes": {
      "unique_id": "subscriber-uuid-123",
      "name": "Alice M. Johnson",
      "status": "active",
      "updated_at": "2026-02-15T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Subscriber not found

---

### DELETE /subscribers/:unique_id/ - Delete Subscriber

Deletes a subscriber.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/subscribers/subscriber-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Subscriber not found

---

### POST /communications/unsubscribe - Unsubscribe

Processes a communication unsubscribe request.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/communications/unsubscribe" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "subscriber@example.com",
    "reason": "no_longer_interested"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Email to unsubscribe |
| `reason` | string | No | Unsubscribe reason |

**Response 200:**
```json
{
  "message": "Successfully unsubscribed"
}
```

**Errors:**
- `404 Not Found` - Email not found in subscriber list

---

## Data Models

### Subscriber
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | Subscriber email address |
| `name` | string | Subscriber name |
| `source` | string | Subscription source |
| `subscribed_at` | timestamp | When the subscription started |
| `status` | enum | active, unsubscribed, bounced |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Subscriber Not Found",
    "detail": "The requested subscriber could not be found."
  }]
}
```
