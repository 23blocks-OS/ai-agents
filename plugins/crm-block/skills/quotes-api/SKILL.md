---
name: crm-quotes-api
description: Manage 23blocks CRM quotes via REST API. Use when creating, updating, or tracking sales quotes with approval workflows and status management.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# CRM Quotes API

Complete API reference for 23blocks CRM quote management with approval workflows.

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
curl -X GET "$BLOCKS_API_URL/quotes/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /quotes/ - List Quotes

Lists all quotes with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/quotes/?page=1&records=20" \
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
      "id": "quote-uuid-123",
      "type": "quote",
      "attributes": {
        "unique_id": "quote-uuid-123",
        "account_id": "account-uuid-456",
        "total": 25000.00,
        "valid_until": "2026-03-31",
        "terms": "Net 30",
        "status": "sent",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 18
  }
}
```

---

### GET /quotes/:unique_id - Get Quote

Retrieves a single quote by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/quotes/quote-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "quote-uuid-123",
    "type": "quote",
    "attributes": {
      "unique_id": "quote-uuid-123",
      "account_id": "account-uuid-456",
      "total": 25000.00,
      "valid_until": "2026-03-31",
      "terms": "Net 30",
      "status": "sent",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    },
    "relationships": {
      "account": {
        "data": { "id": "account-uuid-456", "type": "account" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Quote not found

---

### POST /quotes/ - Create Quote

Creates a new quote.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/quotes/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "quote": {
      "account_id": "account-uuid-456",
      "total": 50000.00,
      "valid_until": "2026-06-30",
      "terms": "Net 30, 2% discount for early payment"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | uuid | Yes | Associated account ID |
| `total` | decimal | Yes | Quote total amount |
| `valid_until` | date | No | Quote expiration date |
| `terms` | string | No | Payment terms and conditions |

**Response 201:**
```json
{
  "data": {
    "id": "quote-uuid-456",
    "type": "quote",
    "attributes": {
      "unique_id": "quote-uuid-456",
      "account_id": "account-uuid-456",
      "total": 50000.00,
      "valid_until": "2026-06-30",
      "terms": "Net 30, 2% discount for early payment",
      "status": "draft",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /quotes/:unique_id/ - Update Quote

Updates an existing quote.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/quotes/quote-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "quote": {
      "total": 27500.00,
      "status": "sent"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "quote-uuid-123",
    "type": "quote",
    "attributes": {
      "unique_id": "quote-uuid-123",
      "total": 27500.00,
      "status": "sent",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Quote not found

---

### DELETE /quotes/:unique_id/ - Delete Quote

Deletes a quote.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/quotes/quote-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Quote not found

---

## Data Models

### Quote
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `account_id` | uuid | Associated account ID |
| `total` | decimal | Quote total amount |
| `valid_until` | date | Quote expiration date |
| `terms` | string | Payment terms and conditions |
| `status` | enum | draft, sent, accepted, rejected |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Quote Not Found",
    "detail": "The requested quote could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-crm
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
// QuotesService — client.crm.quotes
list(params?: ListQuotesParams): Promise<PageResult<Quote>>;
get(uniqueId: string): Promise<Quote>;
create(data: CreateQuoteRequest): Promise<Quote>;
update(uniqueId: string, data: UpdateQuoteRequest): Promise<Quote>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<Quote>;
search(query: string, params?: ListQuotesParams): Promise<PageResult<Quote>>;
listDeleted(params?: ListQuotesParams): Promise<PageResult<Quote>>;
```

### TypeScript Types

```typescript
import type {
  Quote,
  CreateQuoteRequest,
  UpdateQuoteRequest,
  ListQuotesParams,
} from '@23blocks/block-crm';
```

### React Hook

```typescript
import { useCrmBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCrmBlock();

  // Example: List all quotes
  const result = await client.crm.quotes.list();
}
```
