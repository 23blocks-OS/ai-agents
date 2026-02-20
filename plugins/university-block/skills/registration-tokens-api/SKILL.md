---
name: university-registration-tokens-api
description: Manage 23blocks university registration tokens via REST API. Admin-only endpoints for listing, viewing, updating, and deleting course registration tokens with usage tracking.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Registration Tokens API

Complete API reference for 23blocks university registration token management. All endpoints are admin-only.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://university.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | University API base URL | `https://university.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/tokens" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

> **Note:** All registration token endpoints require admin-level authorization.

---

## Endpoints

### GET /tokens/ - List Tokens

Lists all registration tokens with pagination and search.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tokens?page=1&records=20&search=FALL2025" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Records per page (default: 20) |
| `search` | string | No | Search tokens by value or user |

**Response 200:**
```json
{
  "data": [
    {
      "id": "token-uuid-123",
      "type": "registration_token",
      "attributes": {
        "unique_id": "token-uuid-123",
        "token": "FALL2025-CS101-A1B2C3",
        "user_unique_id": null,
        "course_id": "course-uuid-789",
        "expires_at": "2025-09-01T00:00:00Z",
        "max_uses": 30,
        "current_uses": 12,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "token-uuid-456",
      "type": "registration_token",
      "attributes": {
        "unique_id": "token-uuid-456",
        "token": "SPRING2025-MATH201-X9Y8Z7",
        "user_unique_id": "user-uuid-111",
        "course_id": "course-uuid-222",
        "expires_at": "2025-02-15T00:00:00Z",
        "max_uses": 1,
        "current_uses": 1,
        "status": "used",
        "created_at": "2025-01-05T08:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 52
  }
}
```

---

### GET /tokens/:unique_id/ - Get Token

Retrieves a specific registration token.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tokens/token-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "token-uuid-123",
    "type": "registration_token",
    "attributes": {
      "unique_id": "token-uuid-123",
      "token": "FALL2025-CS101-A1B2C3",
      "user_unique_id": null,
      "course_id": "course-uuid-789",
      "expires_at": "2025-09-01T00:00:00Z",
      "max_uses": 30,
      "current_uses": 12,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Token not found
- `403 Forbidden` - Admin access required

---

### PUT /tokens/:unique_id/ - Update Token

Updates an existing registration token.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/tokens/token-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "registration_token": {
      "max_uses": 50,
      "expires_at": "2025-10-01T00:00:00Z",
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | string | No | Token value |
| `user_unique_id` | uuid | No | Assigned user ID |
| `course_id` | uuid | No | Associated course ID |
| `expires_at` | datetime | No | Expiration date (ISO 8601) |
| `max_uses` | integer | No | Maximum allowed uses |
| `status` | enum | No | active, used, expired, revoked |

**Response 200:**
```json
{
  "data": {
    "id": "token-uuid-123",
    "type": "registration_token",
    "attributes": {
      "unique_id": "token-uuid-123",
      "token": "FALL2025-CS101-A1B2C3",
      "user_unique_id": null,
      "course_id": "course-uuid-789",
      "expires_at": "2025-10-01T00:00:00Z",
      "max_uses": 50,
      "current_uses": 12,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Token not found
- `403 Forbidden` - Admin access required
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /tokens/:unique_id/ - Delete Token

Deletes a registration token.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/tokens/token-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Token not found
- `403 Forbidden` - Admin access required

---

## Data Models

### RegistrationToken
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `token` | string | Token value |
| `user_unique_id` | uuid | Assigned user ID (null for multi-use tokens) |
| `course_id` | uuid | Associated course ID |
| `expires_at` | datetime | Expiration date |
| `max_uses` | integer | Maximum allowed uses |
| `current_uses` | integer | Number of times used |
| `status` | enum | active, used, expired, revoked |
| `created_at` | timestamp | Creation time |

### Token Statuses
| Status | Description |
|--------|-------------|
| `active` | Token is valid and can be used |
| `used` | Token has reached max uses |
| `expired` | Token has passed its expiration date |
| `revoked` | Token has been manually revoked by admin |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "403",
    "code": "forbidden",
    "title": "Forbidden",
    "detail": "Admin access required to manage registration tokens."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-university
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
// RegistrationTokensService — client.university.registrationTokens
client.university.registrationTokens.list(params?: ListRegistrationTokensParams): Promise<PageResult<RegistrationToken>>;
client.university.registrationTokens.get(uniqueId: string): Promise<RegistrationToken>;
client.university.registrationTokens.create(data: CreateRegistrationTokenRequest): Promise<RegistrationToken>;
client.university.registrationTokens.update(uniqueId: string, data: UpdateRegistrationTokenRequest): Promise<RegistrationToken>;
client.university.registrationTokens.delete(uniqueId: string): Promise<void>;
client.university.registrationTokens.validate(tokenCode: string): Promise<TokenValidationResult>;
client.university.registrationTokens.use(tokenCode: string, userUniqueId: string): Promise<{ success: boolean; enrollmentUniqueId?: string; error?: string }>;
client.university.registrationTokens.revoke(uniqueId: string): Promise<RegistrationToken>;
client.university.registrationTokens.generateBatch(request: CreateRegistrationTokenRequest & { count: number }): Promise<RegistrationToken[]>;
```

### TypeScript Types

```typescript
import type {
  RegistrationToken,
  CreateRegistrationTokenRequest,
  UpdateRegistrationTokenRequest,
  ListRegistrationTokensParams,
  TokenValidationResult,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.registrationTokens.list();
}
```
