---
name: 23blocks-university-registration-tokens-api
description: "University Block course registration tokens and usage. Use when managing enrollment codes. Admin only."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Registration Tokens API

Complete API reference for 23blocks university registration token management. All endpoints are admin-only.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://university.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /tokens/ - List Tokens

Lists all registration tokens with pagination and search.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tokens?page=1&size=20&search=FALL2025" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `size` | integer | No | Items per page (default: 15, max: 100) |
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"403","code":"forbidden","title":"Forbidden","detail":"Admin access required to manage registration tokens."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-university`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useUniversityBlock()` from `@23blocks/react`.

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
