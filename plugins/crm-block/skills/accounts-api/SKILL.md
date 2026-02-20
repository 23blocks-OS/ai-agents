---
name: crm-accounts-api
description: Manage 23blocks CRM accounts via REST API. Use when creating, updating, or managing customer accounts including details, logos, documents, leads, contacts, meetings, locations, categories, and quotes.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# CRM Accounts API

Complete API reference for 23blocks CRM account management with details, documents, leads, contacts, meetings, locations, categories, and quotes.

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
curl -X GET "$BLOCKS_API_URL/accounts/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /accounts/ - List Accounts

Lists all accounts with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/accounts/?page=1&records=20" \
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
      "id": "account-uuid-123",
      "type": "account",
      "attributes": {
        "unique_id": "account-uuid-123",
        "name": "Acme Corp",
        "tax_id": "12-3456789",
        "industry": "Technology",
        "website": "https://acme.com",
        "phone": "+1-555-0100",
        "email": "info@acme.com",
        "address": "123 Main St, San Francisco, CA",
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### GET /accounts/:unique_id - Get Account

Retrieves a single account by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/accounts/account-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "account-uuid-123",
    "type": "account",
    "attributes": {
      "unique_id": "account-uuid-123",
      "name": "Acme Corp",
      "tax_id": "12-3456789",
      "industry": "Technology",
      "website": "https://acme.com",
      "phone": "+1-555-0100",
      "email": "info@acme.com",
      "address": "123 Main St, San Francisco, CA",
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    },
    "relationships": {
      "contacts": {
        "data": [
          { "id": "contact-uuid", "type": "contact" }
        ]
      },
      "leads": {
        "data": []
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Account not found

---

### GET /accounts/status/trash - Trashed Accounts

Lists all trashed (soft-deleted) accounts.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/accounts/status/trash" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Same format as List Accounts, filtered by trash status.

---

### GET /accounts/status/archive - Archived Accounts

Lists all archived accounts.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/accounts/status/archive" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Same format as List Accounts, filtered by archive status.

---

### POST /accounts/validate - Validate Account

Validates account data before creation.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/accounts/validate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account": {
      "name": "Acme Corp",
      "email": "info@acme.com"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "valid": true,
    "errors": []
  }
}
```

---

### POST /accounts/validate_tax_id - Validate Tax ID

Validates a tax ID for an account.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/accounts/validate_tax_id" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tax_id": "12-3456789"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tax_id` | string | Yes | Tax ID to validate |

**Response 200:**
```json
{
  "data": {
    "valid": true,
    "tax_id": "12-3456789"
  }
}
```

---

### POST /accounts/ - Create Account

Creates a new customer account.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/accounts/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account": {
      "name": "Acme Corp",
      "tax_id": "12-3456789",
      "industry": "Technology",
      "website": "https://acme.com",
      "phone": "+1-555-0100",
      "email": "info@acme.com",
      "address": "123 Main St, San Francisco, CA"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Account name |
| `tax_id` | string | No | Tax identification number |
| `industry` | string | No | Industry sector |
| `website` | string | No | Website URL |
| `phone` | string | No | Phone number |
| `email` | string | No | Email address |
| `address` | string | No | Physical address |

**Response 201:**
```json
{
  "data": {
    "id": "account-uuid-123",
    "type": "account",
    "attributes": {
      "unique_id": "account-uuid-123",
      "name": "Acme Corp",
      "tax_id": "12-3456789",
      "industry": "Technology",
      "website": "https://acme.com",
      "phone": "+1-555-0100",
      "email": "info@acme.com",
      "address": "123 Main St, San Francisco, CA",
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /accounts/:unique_id/ - Update Account

Updates an existing account.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/accounts/account-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account": {
      "name": "Acme Corporation",
      "phone": "+1-555-0200"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "account-uuid-123",
    "type": "account",
    "attributes": {
      "unique_id": "account-uuid-123",
      "name": "Acme Corporation",
      "phone": "+1-555-0200",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Account not found

---

### DELETE /accounts/:unique_id/ - Delete Account

Soft-deletes an account (moves to trash).

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/accounts/account-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Account not found

---

### DELETE /accounts/:unique_id/archive - Archive Account

Archives an account instead of deleting it.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/accounts/account-uuid-123/archive" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "account-uuid-123",
    "type": "account",
    "attributes": {
      "status": "archived",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

---

## Account Details

### POST /accounts/:unique_id/details - Add Details

Adds additional details to an account.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/accounts/account-uuid-123/details" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "detail": {
      "key": "annual_revenue",
      "value": "5000000"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | string | Yes | Detail key name |
| `value` | string | Yes | Detail value |

**Response 201:**
```json
{
  "data": {
    "id": "detail-uuid",
    "type": "account_detail",
    "attributes": {
      "key": "annual_revenue",
      "value": "5000000",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /accounts/:unique_id/details - Update Details

Updates existing account details.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/accounts/account-uuid-123/details" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "detail": {
      "key": "annual_revenue",
      "value": "7500000"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "detail-uuid",
    "type": "account_detail",
    "attributes": {
      "key": "annual_revenue",
      "value": "7500000",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

---

## Account Logo

### PUT /accounts/:unique_id/presign_logo - Presign Logo

Gets a presigned URL for uploading an account logo.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/accounts/account-uuid-123/presign_logo" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "filename": "logo.png",
      "content_type": "image/png"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "presigned_url": "https://s3.amazonaws.com/bucket/path?signature=...",
    "file_key": "accounts/account-uuid-123/logo.png"
  }
}
```

---

### PUT /accounts/:unique_id/logo/publish - Publish Logo

Publishes the uploaded logo for the account.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/accounts/account-uuid-123/logo/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file_key": "accounts/account-uuid-123/logo.png"
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "account-uuid-123",
    "type": "account",
    "attributes": {
      "logo_url": "https://cdn.23blocks.com/accounts/account-uuid-123/logo.png"
    }
  }
}
```

---

## Account Leads

### GET /accounts/:unique_id/leads - Account Leads

Lists all leads associated with an account.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/accounts/account-uuid-123/leads" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "lead-uuid-456",
      "type": "lead",
      "attributes": {
        "unique_id": "lead-uuid-456",
        "name": "Enterprise Deal",
        "source": "website",
        "value": 50000,
        "stage": "qualified",
        "status": "active"
      }
    }
  ]
}
```

---

### POST /accounts/:unique_id/leads - Add Lead to Account

Associates a lead with an account.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/accounts/account-uuid-123/leads" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "lead": {
      "name": "New Enterprise Deal",
      "source": "referral",
      "value": 75000,
      "stage": "new"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "lead-uuid-789",
    "type": "lead",
    "attributes": {
      "unique_id": "lead-uuid-789",
      "name": "New Enterprise Deal",
      "account_id": "account-uuid-123",
      "source": "referral",
      "value": 75000,
      "stage": "new",
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

## Account Contacts

### GET /accounts/:unique_id/contacts - Account Contacts

Lists all contacts associated with an account.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/accounts/account-uuid-123/contacts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "contact-uuid-456",
      "type": "contact",
      "attributes": {
        "unique_id": "contact-uuid-456",
        "first_name": "Jane",
        "last_name": "Smith",
        "email": "jane@acme.com",
        "phone": "+1-555-0101",
        "title": "VP Sales",
        "status": "active"
      }
    }
  ]
}
```

---

### GET /accounts/:unique_id/contacts/:contact_unique_id - Get Specific Contact

Retrieves a specific contact associated with an account.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/accounts/account-uuid-123/contacts/contact-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "contact-uuid-456",
    "type": "contact",
    "attributes": {
      "unique_id": "contact-uuid-456",
      "first_name": "Jane",
      "last_name": "Smith",
      "email": "jane@acme.com",
      "phone": "+1-555-0101",
      "title": "VP Sales",
      "account_id": "account-uuid-123",
      "status": "active"
    }
  }
}
```

---

### POST /accounts/:unique_id/contacts - Add Contact to Account

Associates a contact with an account.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/accounts/account-uuid-123/contacts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contact": {
      "first_name": "Bob",
      "last_name": "Wilson",
      "email": "bob@acme.com",
      "title": "CTO"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "contact-uuid-789",
    "type": "contact",
    "attributes": {
      "unique_id": "contact-uuid-789",
      "first_name": "Bob",
      "last_name": "Wilson",
      "email": "bob@acme.com",
      "title": "CTO",
      "account_id": "account-uuid-123",
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /accounts/:unique_id/contacts/:contact_unique_id - Remove Contact

Removes a contact association from an account.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/accounts/account-uuid-123/contacts/contact-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Account Meetings

### GET /accounts/:unique_id/meetings - Account Meetings

Lists all meetings associated with an account.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/accounts/account-uuid-123/meetings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "meeting-uuid-789",
      "type": "meeting",
      "attributes": {
        "unique_id": "meeting-uuid-789",
        "title": "Quarterly Review",
        "start_time": "2026-03-15T14:00:00Z",
        "end_time": "2026-03-15T15:00:00Z",
        "status": "scheduled"
      }
    }
  ]
}
```

---

## Account Documents

### GET /accounts/:unique_id/documents - List Documents

Lists all documents associated with an account.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/accounts/account-uuid-123/documents" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "doc-uuid-123",
      "type": "document",
      "attributes": {
        "unique_id": "doc-uuid-123",
        "filename": "contract.pdf",
        "content_type": "application/pdf",
        "file_size": 245000,
        "url": "https://cdn.23blocks.com/documents/contract.pdf",
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### PUT /accounts/:unique_id/presign_document - Presign Document

Gets a presigned URL for uploading a document.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/accounts/account-uuid-123/presign_document" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "filename": "proposal.pdf",
      "content_type": "application/pdf"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "presigned_url": "https://s3.amazonaws.com/bucket/path?signature=...",
    "file_key": "accounts/account-uuid-123/documents/proposal.pdf"
  }
}
```

---

### POST /accounts/:unique_id/documents - Upload Document

Registers an uploaded document with an account.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/accounts/account-uuid-123/documents" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "document": {
      "filename": "proposal.pdf",
      "file_key": "accounts/account-uuid-123/documents/proposal.pdf",
      "content_type": "application/pdf"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "doc-uuid-456",
    "type": "document",
    "attributes": {
      "unique_id": "doc-uuid-456",
      "filename": "proposal.pdf",
      "content_type": "application/pdf",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /accounts/:unique_id/documents/:unique_document_id - Delete Document

Deletes a document from an account.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/accounts/account-uuid-123/documents/doc-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Account Categories

### POST /accounts/:unique_id/categories - Assign Category

Assigns a category to an account.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/accounts/account-uuid-123/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Enterprise"
    }
  }'
```

**Response 201:**
```json
{
  "message": "Category assigned successfully"
}
```

---

## Account Locations

### GET /accounts/:unique_id/locations - List Locations

Lists all locations for an account.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/accounts/account-uuid-123/locations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "location-uuid",
      "type": "location",
      "attributes": {
        "unique_id": "location-uuid",
        "name": "Headquarters",
        "address": "123 Main St",
        "city": "San Francisco",
        "state": "CA",
        "zip": "94102",
        "country": "US"
      }
    }
  ]
}
```

---

### POST /accounts/:unique_id/locations - Add Location

Adds a location to an account.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/accounts/account-uuid-123/locations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "location": {
      "name": "Branch Office",
      "address": "456 Oak Ave",
      "city": "Los Angeles",
      "state": "CA",
      "zip": "90001",
      "country": "US"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Location name |
| `address` | string | No | Street address |
| `city` | string | No | City |
| `state` | string | No | State/Province |
| `zip` | string | No | ZIP/Postal code |
| `country` | string | No | Country code |

**Response 201:**
```json
{
  "data": {
    "id": "location-uuid-2",
    "type": "location",
    "attributes": {
      "unique_id": "location-uuid-2",
      "name": "Branch Office",
      "address": "456 Oak Ave",
      "city": "Los Angeles",
      "state": "CA",
      "zip": "90001",
      "country": "US",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

## Account Quotes

### GET /accounts/:unique_id/quotes - Account Quotes

Lists all quotes for an account.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/accounts/account-uuid-123/quotes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "quote-uuid",
      "type": "quote",
      "attributes": {
        "unique_id": "quote-uuid",
        "total": 25000.00,
        "valid_until": "2026-03-31",
        "status": "sent"
      }
    }
  ]
}
```

---

### POST /accounts/:unique_id/quotes - Add Quote

Creates a quote for an account.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/accounts/account-uuid-123/quotes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "quote": {
      "total": 50000.00,
      "valid_until": "2026-06-30",
      "terms": "Net 30"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "quote-uuid-2",
    "type": "quote",
    "attributes": {
      "unique_id": "quote-uuid-2",
      "account_id": "account-uuid-123",
      "total": 50000.00,
      "valid_until": "2026-06-30",
      "terms": "Net 30",
      "status": "draft",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

## Data Models

### Account
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Account name |
| `tax_id` | string | Tax identification number |
| `industry` | string | Industry sector |
| `website` | string | Website URL |
| `phone` | string | Phone number |
| `email` | string | Email address |
| `address` | string | Physical address |
| `status` | enum | active, archived, trash |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Location
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Location name |
| `address` | string | Street address |
| `city` | string | City |
| `state` | string | State/Province |
| `zip` | string | ZIP/Postal code |
| `country` | string | Country code |

### Document
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `filename` | string | File name |
| `content_type` | string | MIME type |
| `file_size` | integer | File size in bytes |
| `url` | string | Download URL |
| `created_at` | timestamp | Upload time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Account Not Found",
    "detail": "The requested account could not be found."
  }]
}
```
