---
name: 23blocks-auth-invitations-api
description: "Auth Block user invitations: invite by email, resend, accept (set password). Use for invite-to-sign-up workflows."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.1"
---

# Invitations API

Complete API reference for 23blocks user invitations. An invitation creates a user in `invited` status and emails them a link; the invitee accepts by setting a password. There is no separate invitation resource: the response is the invited **user**.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://auth.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### POST /auth/invitation - Invite User

Invites a user into the caller's company. Requires scope `users:write`. The invitation always goes into the company the token and `X-API-KEY` belong to.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/invitation" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newmember@example.com",
      "name": "New Member",
      "role_id": 3,
      "accept_invitation_url": "https://app.example.com/accept-invite"
    }
  }'
```

**Request Parameters (under `user`):**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Invitee email |
| `name` | string | No | Invitee display name |
| `role_id` | integer | No | Role to assign. Must be a role the inviter may assign (see below). Defaults to the tenant's `user` role |
| `accept_invitation_url` | string | No | Link target used in the invitation email |
| `unique_id` | string | No | Unique ID for the new user (generated if omitted) |
| `appid` | string | No | Must match the company of the authenticated app; a mismatch is rejected with `403`. Omit it |

**Role rules:** the inviter can only grant a role whose privilege level is not higher than their own, and protected roles only if the inviter is a super admin. Otherwise the call returns `403 ROLE_NOT_ASSIGNABLE`.

**Response 201:** the invited user, status `invited`.

**Behavior for an existing email:**
- Active or already-registered user: `422` code `10112` (User Already Registered).
- Pending invitation, not expired: the invitation is re-sent automatically and `201` is returned.
- Pending invitation, expired: `422` code `10113`. Use the resend endpoint.

**Errors:**
- `400` `INVALID_APPID` - No company resolved for the app, or the `appid` in the body is invalid
- `403` `APPID_MISMATCH` - Body `appid` belongs to a different company than the authenticated app
- `403` `ROLE_NOT_ASSIGNABLE` - `role_id` is not a role the inviter can assign
- `403` - Token lacks the `users:write` scope
- `422` `INVALID_EMAIL` - `email` missing

---

### POST /auth/invitation/resend - Resend Invitation

Sends a fresh invitation email to a user who has not accepted yet. Works for both expired and unexpired invitations.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/auth/invitation/resend" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "invitation": {
      "email": "newmember@example.com",
      "accept_invitation_url": "https://app.example.com/accept-invite"
    }
  }'
```

**Request Parameters (under `invitation`):**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Email of the invited user |
| `accept_invitation_url` | string | No | Link target used in the invitation email |
| `appid` | string | No | Company app ID |

**Response 201:** the invited user.

**Errors:**
- `400` `INVALID_APPID` - App ID invalid or deactivated
- `422` `INVALID_EMAIL` - `email` missing
- `422` `10114` - No user with this email
- `422` `10112` - User already registered (invitation already accepted)

---

### GET /auth/invitation/accept - Follow Invitation Link

The link in the invitation email. Redirects to `redirect_url` with `invitation_token` and `appid` as query parameters so the app can show its own "set password" screen. It does not accept the invitation by itself.

**Query Parameters:** `invitation_token`, `appid`, `redirect_url`.

An invalid token redirects to `{redirect_url}/auth/invitation/error`.

---

### PUT /auth/invitation - Accept Invitation

Completes the invitation: sets the password, activates the user, creates the profile, and returns a login token. No bearer token is needed; the invitation token is the credential.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/auth/invitation" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "invitation": {
      "invitation_token": "token-from-email-link",
      "name": "New Member",
      "user_name": "newmember",
      "password": "a-strong-password",
      "phone_number": "+15551234567",
      "preferred_language": "en",
      "time_zone": "America/Denver"
    }
  }'
```

**Request Parameters (under `invitation`):**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `invitation_token` | string | Yes | Token from the invitation link |
| `password` | string | Yes | Password to set |
| `name` | string | Yes | Full name (split into first, middle, last for the profile) |
| `user_name` | string | No | Username |
| `phone_number` | string | No | Phone number |
| `preferred_language` | string | No | Language code (default `en`) |
| `time_zone` | string | No | Time zone (default `UTC`) |
| `subscription` | string | No | Subscription model unique ID to subscribe the user to |

**Response 200:** the user, plus `meta.auth` with a ready-to-use session:
```json
{
  "data": { "id": "user-uuid", "type": "user", "attributes": { "email": "newmember@example.com", "status": "active" } },
  "meta": {
    "auth": {
      "access_token": "eyJhbGciOiJSUzI1NiIs...",
      "token_type": "Bearer",
      "expires_in": 3600,
      "expires_at": "2026-10-21T11:30:00Z",
      "scope": "users:read files:read"
    }
  }
}
```

**Errors:**
- `422` `10111` - Invalid or expired token, or password rejected

---

## Notes

- **Scope**: sending an invitation needs `users:write` (previously any authenticated user could call it).
- **Tenant**: the target company is taken from the token and `X-API-KEY`. Do not send `appid` in the body; a different company than the authenticated one returns `403`.
- **Role**: `role_id` is checked against the inviter's own privilege level. Leave it out to get the default user role.
- **No cancel/decline/list endpoints**: earlier versions of this skill documented `/invitations` CRUD with accept, decline and cancel. Those routes do not exist in the Auth API; use the endpoints above.
- **Organization membership** is a separate concept: see `23blocks-auth-org-members-api`. For group membership see `23blocks-auth-groups-api`.

## Data Models

An invited user is a normal user with `status` `invited` until the invitation is accepted, then `active`. See `23blocks-auth-users-api` for the user fields.

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"403","source":"Invitation Controller - AUTH API","code":"ROLE_NOT_ASSIGNABLE","title":"Role Not Assignable","detail":"..."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-authentication`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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

If the SDK still sends `appid` or a `roleId` the inviter cannot assign, it will now get a `403`; check the installed SDK version.
