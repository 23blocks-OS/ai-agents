---
name: auth-invitations-api
description: Manage 23blocks invitations via REST API. Use when creating, sending, resending, accepting, declining, or cancelling invitations to organizations.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Invitations API

Complete API reference for 23blocks invitation workflow management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://auth.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Auth API base URL | `https://auth.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/invitations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /invitations - List Invitations

Lists all invitations.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/invitations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `status` | string | No | Filter by status (pending, accepted, declined, expired) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "invite-uuid-123",
      "type": "invitation",
      "attributes": {
        "unique_id": "invite-uuid-123",
        "email": "invitee@example.com",
        "status": "pending",
        "role": "member",
        "sent_at": "2025-01-10T10:30:00Z",
        "expires_at": "2025-01-17T10:30:00Z",
        "created_at": "2025-01-10T10:30:00Z"
      },
      "relationships": {
        "organization": {
          "data": { "id": "org-uuid", "type": "organization" }
        },
        "inviter": {
          "data": { "id": "user-uuid", "type": "user" }
        }
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 5
  }
}
```

---

### GET /invitations/:unique_id - Get Invitation

Retrieves a single invitation by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/invitations/invite-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "invite-uuid-123",
    "type": "invitation",
    "attributes": {
      "unique_id": "invite-uuid-123",
      "email": "invitee@example.com",
      "status": "pending",
      "role": "member",
      "message": "Join our team!",
      "sent_at": "2025-01-10T10:30:00Z",
      "expires_at": "2025-01-17T10:30:00Z",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Invitation not found

---

### POST /invitations - Create Invitation

Creates a new invitation.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/invitations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "invitation": {
      "email": "newmember@example.com",
      "role": "member",
      "organization_id": "org-uuid-123",
      "message": "We would love to have you on our team!"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Invitee email address |
| `role` | string | No | Role to assign (default: member) |
| `organization_id` | uuid | Yes | Target organization |
| `message` | string | No | Personal invitation message |

**Response 201:**
```json
{
  "data": {
    "id": "new-invite-uuid",
    "type": "invitation",
    "attributes": {
      "unique_id": "new-invite-uuid",
      "email": "newmember@example.com",
      "status": "pending",
      "role": "member",
      "message": "We would love to have you on our team!",
      "expires_at": "2025-01-19T10:30:00Z",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - User already invited or is a member
- `422 Unprocessable Entity` - Validation errors

---

### PUT /invitations/:unique_id - Update Invitation

Updates a pending invitation.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/invitations/invite-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "invitation": {
      "role": "admin",
      "message": "Updated invitation message"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "invite-uuid-123",
    "type": "invitation",
    "attributes": {
      "unique_id": "invite-uuid-123",
      "role": "admin",
      "message": "Updated invitation message",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /invitations/:unique_id - Delete Invitation

Deletes an invitation.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/invitations/invite-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### POST /invitations/:unique_id/send - Send Invitation

Sends the invitation email to the invitee.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/invitations/invite-uuid-123/send" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "invite-uuid-123",
    "type": "invitation",
    "attributes": {
      "unique_id": "invite-uuid-123",
      "status": "pending",
      "sent_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /invitations/:unique_id/resend - Resend Invitation

Resends the invitation email.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/invitations/invite-uuid-123/resend" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "invite-uuid-123",
    "type": "invitation",
    "attributes": {
      "unique_id": "invite-uuid-123",
      "status": "pending",
      "resent_at": "2025-01-12T14:00:00Z",
      "resend_count": 2
    }
  }
}
```

---

### POST /invitations/:unique_id/accept - Accept Invitation

Accepts an invitation and joins the organization.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/invitations/invite-uuid-123/accept" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "invite-uuid-123",
    "type": "invitation",
    "attributes": {
      "unique_id": "invite-uuid-123",
      "status": "accepted",
      "accepted_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Invitation not found
- `422 Unprocessable Entity` - Invitation expired or already processed

---

### POST /invitations/:unique_id/decline - Decline Invitation

Declines an invitation.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/invitations/invite-uuid-123/decline" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "invite-uuid-123",
    "type": "invitation",
    "attributes": {
      "unique_id": "invite-uuid-123",
      "status": "declined",
      "declined_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /invitations/:unique_id/cancel - Cancel Invitation

Cancels a pending invitation.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/invitations/invite-uuid-123/cancel" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "invite-uuid-123",
    "type": "invitation",
    "attributes": {
      "unique_id": "invite-uuid-123",
      "status": "cancelled",
      "cancelled_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Data Models

### Invitation
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | Invitee email |
| `status` | enum | pending, accepted, declined, cancelled, expired |
| `role` | string | Role to assign upon acceptance |
| `message` | string | Personal invitation message |
| `organization_id` | uuid | Target organization |
| `inviter_id` | uuid | User who created invitation |
| `sent_at` | timestamp | When email was sent |
| `expires_at` | timestamp | Invitation expiration |
| `accepted_at` | timestamp | When accepted |
| `declined_at` | timestamp | When declined |
| `cancelled_at` | timestamp | When cancelled |
| `resend_count` | integer | Number of resends |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "invitation_expired",
    "title": "Invitation Expired",
    "detail": "This invitation has expired and can no longer be accepted."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-authentication
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
// AuthService (invitation methods) — client.authentication.auth
sendInvitation(request: InvitationRequest): Promise<void>;
acceptInvitation(request: AcceptInvitationRequest): Promise<SignInResponse>;
resendInvitation(request: ResendInvitationRequest): Promise<User>;
```

### TypeScript Types

```typescript
import type {
  InvitationRequest,
  AcceptInvitationRequest,
  ResendInvitationRequest,
  SignInResponse,
  User,
} from '@23blocks/block-authentication';
```

### React Hook

```typescript
import { useAuthenticationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAuthenticationBlock();

  // Example: Send an invitation
  await client.authentication.auth.sendInvitation({
    email: 'newmember@example.com',
    roleId: 'role-uuid',
    redirectUrl: 'https://app.example.com/accept-invite',
  });
}
```
