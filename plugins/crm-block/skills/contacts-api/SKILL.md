---
name: crm-contacts-api
description: Manage 23blocks CRM contacts via REST API. Use when creating, updating, archiving contacts, managing contact profiles, history, and documents.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# CRM Contacts API

Complete API reference for 23blocks CRM contact management with profiles, history tracking, and document handling.

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
curl -X GET "$BLOCKS_API_URL/contacts/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /contacts/ - List Contacts

Lists all contacts with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/contacts/?page=1&records=20" \
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
      "id": "contact-uuid-123",
      "type": "contact",
      "attributes": {
        "unique_id": "contact-uuid-123",
        "first_name": "Jane",
        "last_name": "Smith",
        "email": "jane@example.com",
        "phone": "+1-555-0101",
        "account_id": "account-uuid-456",
        "title": "VP Sales",
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 45
  }
}
```

---

### GET /contacts/:unique_id - Get Contact

Retrieves a single contact by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/contacts/contact-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "contact-uuid-123",
    "type": "contact",
    "attributes": {
      "unique_id": "contact-uuid-123",
      "first_name": "Jane",
      "last_name": "Smith",
      "email": "jane@example.com",
      "phone": "+1-555-0101",
      "account_id": "account-uuid-456",
      "title": "VP Sales",
      "status": "active",
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
- `404 Not Found` - Contact not found

---

### GET /contacts/trash/show - Trashed Contacts

Lists all trashed (soft-deleted) contacts.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/contacts/trash/show" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Same format as List Contacts, filtered by trash status.

---

### POST /contacts/ - Create Contact

Creates a new contact.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/contacts/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contact": {
      "first_name": "Bob",
      "last_name": "Wilson",
      "email": "bob@example.com",
      "phone": "+1-555-0102",
      "account_id": "account-uuid-456",
      "title": "CTO"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `first_name` | string | Yes | First name |
| `last_name` | string | Yes | Last name |
| `email` | string | Yes | Email address |
| `phone` | string | No | Phone number |
| `account_id` | uuid | No | Associated account ID |
| `title` | string | No | Job title |

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
      "email": "bob@example.com",
      "phone": "+1-555-0102",
      "account_id": "account-uuid-456",
      "title": "CTO",
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### POST /contacts/:unique_id/history - Add History

Adds a history entry to a contact's record.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/contacts/contact-uuid-123/history" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "history": {
      "action": "phone_call",
      "description": "Discussed Q1 contract renewal",
      "occurred_at": "2026-02-15T10:00:00Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | Type of history entry |
| `description` | string | No | Description of the interaction |
| `occurred_at` | timestamp | No | When the interaction occurred |

**Response 201:**
```json
{
  "data": {
    "id": "history-uuid",
    "type": "contact_history",
    "attributes": {
      "action": "phone_call",
      "description": "Discussed Q1 contract renewal",
      "occurred_at": "2026-02-15T10:00:00Z",
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

---

### PUT /contacts/:unique_id/ - Update Contact

Updates an existing contact.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/contacts/contact-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contact": {
      "title": "Senior VP Sales",
      "phone": "+1-555-0200"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "contact-uuid-123",
    "type": "contact",
    "attributes": {
      "unique_id": "contact-uuid-123",
      "title": "Senior VP Sales",
      "phone": "+1-555-0200",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

---

### POST /contacts/:unique_id/profile - Add Profile

Adds a profile to a contact.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/contacts/contact-uuid-123/profile" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "profile": {
      "bio": "20 years experience in enterprise sales",
      "linkedin_url": "https://linkedin.com/in/janesmith",
      "timezone": "America/New_York"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "profile-uuid",
    "type": "contact_profile",
    "attributes": {
      "bio": "20 years experience in enterprise sales",
      "linkedin_url": "https://linkedin.com/in/janesmith",
      "timezone": "America/New_York",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /contacts/:unique_id/profile - Update Profile

Updates a contact's profile.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/contacts/contact-uuid-123/profile" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "profile": {
      "bio": "25 years experience in enterprise sales and partnerships"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "profile-uuid",
    "type": "contact_profile",
    "attributes": {
      "bio": "25 years experience in enterprise sales and partnerships",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /contacts/:unique_id/ - Delete Contact

Soft-deletes a contact (moves to trash).

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/contacts/contact-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Contact not found

---

### DELETE /contacts/:unique_id/archive - Archive Contact

Archives a contact instead of deleting it.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/contacts/contact-uuid-123/archive" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "contact-uuid-123",
    "type": "contact",
    "attributes": {
      "status": "archived",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

---

## Contact Documents

### GET /contacts/:unique_id/documents - List Documents

Lists all documents for a contact.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/contacts/contact-uuid-123/documents" \
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
        "filename": "resume.pdf",
        "content_type": "application/pdf",
        "file_size": 125000,
        "url": "https://cdn.23blocks.com/documents/resume.pdf",
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /contacts/:unique_id/documents - Upload Document

Registers an uploaded document with a contact.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/contacts/contact-uuid-123/documents" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "document": {
      "filename": "nda.pdf",
      "file_key": "contacts/contact-uuid-123/documents/nda.pdf",
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
      "filename": "nda.pdf",
      "content_type": "application/pdf",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /contacts/:unique_id/documents/:unique_document_id - Delete Document

Deletes a document from a contact.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/contacts/contact-uuid-123/documents/doc-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /contacts/:contact_unique_id/presign_document - Presign Document

Gets a presigned URL for uploading a contact document.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/contacts/contact-uuid-123/presign_document" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "filename": "nda.pdf",
      "content_type": "application/pdf"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "presigned_url": "https://s3.amazonaws.com/bucket/path?signature=...",
    "file_key": "contacts/contact-uuid-123/documents/nda.pdf"
  }
}
```

---

## Data Models

### Contact
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `email` | string | Email address |
| `phone` | string | Phone number |
| `account_id` | uuid | Associated account ID |
| `title` | string | Job title |
| `status` | enum | active, archived, trash |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### ContactProfile
| Field | Type | Description |
|-------|------|-------------|
| `bio` | string | Biography |
| `linkedin_url` | string | LinkedIn profile URL |
| `timezone` | string | Timezone |

### ContactHistory
| Field | Type | Description |
|-------|------|-------------|
| `action` | string | Type of history entry |
| `description` | string | Description of interaction |
| `occurred_at` | timestamp | When the interaction occurred |
| `created_at` | timestamp | Record creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Contact Not Found",
    "detail": "The requested contact could not be found."
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
// ContactsService — client.crm.contacts
list(params?: ListContactsParams): Promise<PageResult<Contact>>;
get(uniqueId: string): Promise<Contact>;
create(data: CreateContactRequest): Promise<Contact>;
update(uniqueId: string, data: UpdateContactRequest): Promise<Contact>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<Contact>;
search(query: string, params?: ListContactsParams): Promise<PageResult<Contact>>;
listDeleted(params?: ListContactsParams): Promise<PageResult<Contact>>;
```

### TypeScript Types

```typescript
import type {
  Contact,
  ContactProfile,
  CreateContactRequest,
  UpdateContactRequest,
  ListContactsParams,
} from '@23blocks/block-crm';
```

### React Hook

```typescript
import { useCrmBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCrmBlock();

  // Example: List all contacts
  const result = await client.crm.contacts.list();
}
```
