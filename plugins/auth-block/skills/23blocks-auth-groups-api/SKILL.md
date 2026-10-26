---
name: 23blocks-auth-groups-api
description: "Auth Block groups: shared-plan membership (owner/admin/member), member add/remove, and the group_* JWT claims. Use for family or team subscriptions and group entitlement checks."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Groups API

Complete API reference for 23blocks groups: multi-user membership for shared paid plans (Family, Team). A group has one owner (the purchaser), optional admins and members. A user can belong to several groups. Groups live inside a tenant.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://auth.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Scopes and Access

| Operation | Requires |
|-----------|----------|
| Create group | `groups:write` |
| List own groups (`user_id` / `owner_id` filter) | no special scope |
| List all groups (no filter) | `groups:admin` |
| Show group | any active member, or `groups:admin` |
| Update group | group owner, admin member, or `groups:admin` |
| Delete group | group owner, admin member, or `groups:admin` |
| Add / remove member | group owner, admin member, or `groups:admin` |
| Create / list / revoke invitations | group owner, admin member, or `groups:admin` |
| Accept an invitation | the invited user (their account email must match) |
| `GET /users/:user_id/groups` | a token for that user, or `groups:admin` |

Scopes: `groups:read`, `groups:write`, `groups:delete`, `groups:admin`. Only the admin role holds them by default; every other role must be granted the ones it needs per tenant (for example `groups:write` so purchasers can create groups). A `403` on these endpoints usually means the token lacks the scope or the caller is only a plain member of the group.

## Endpoints

### POST /groups/ - Create Group

Creates a group. The owner is added automatically as a member with `role: owner`.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/groups/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "group": {
      "name": "Smith Family Plan",
      "description": "Family subscription for 5 users"
    }
  }'
```

**Request Parameters (under `group`):**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Group name |
| `description` | string | No | Group description |
| `owner_user_id` | string | No | Owner user unique ID (defaults to the current user; cannot be changed later) |

**Response 201:**
```json
{
  "data": {
    "id": "group-uuid",
    "type": "group",
    "attributes": {
      "name": "Smith Family Plan",
      "description": "Family subscription for 5 users",
      "owner_user_id": "purchaser-uuid",
      "status": "active",
      "member_count": 1
    }
  }
}
```

**Error Codes:**
- `422 Unprocessable Entity` (GW-GRP-001) - Invalid group data

---

### GET /groups/ - List Groups

Returns active groups, paginated (25 per page).

**Request:**
```bash
# Groups a user belongs to
curl -X GET "$BLOCKS_API_URL/groups/?user_id={user_unique_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"

# Groups a user owns
curl -X GET "$BLOCKS_API_URL/groups/?owner_id={user_unique_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string | No | Only groups this user belongs to |
| `owner_id` | string | No | Only groups this user owns |

Without a filter the call needs `groups:admin`.

**Response 200:**
```json
{
  "data": [
    {
      "id": "group-uuid",
      "type": "group",
      "attributes": { "name": "Smith Family Plan", "owner_user_id": "purchaser-uuid" }
    }
  ],
  "meta": { "current_page": 1, "total_pages": 1, "total_count": 1 }
}
```

---

### GET /groups/:id - Show Group

Returns the group with its members included.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/groups/{group_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Error Codes:**
- `404 Not Found` (GW-GRP-404) - Group not found

---

### PUT /groups/:id - Update Group

Updates name and description. The owner cannot be changed.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/groups/{group_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "group": {
      "name": "Updated Name",
      "description": "Updated description"
    }
  }'
```

**Error Codes:**
- `422 Unprocessable Entity` (GW-GRP-002) - Invalid update data
- `404 Not Found` (GW-GRP-404) - Group not found

---

### DELETE /groups/:id - Delete Group

Soft delete: sets `status: deleted` and `enabled: false`. Groups and members are never physically removed.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/groups/{group_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

---

### POST /groups/:id/members - Add Member

Adds an existing user to the group.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/groups/{group_id}/members" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "member-uuid",
    "role": "member"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | string | Yes | Unique ID of the user to add (`user_id` is also accepted) |
| `role` | string | No | `owner`, `admin` or `member` (default `member`) |

**Response 201:**
```json
{
  "data": {
    "id": "1",
    "type": "group_member",
    "attributes": {
      "user_unique_id": "member-uuid",
      "role": "member",
      "status": "active"
    }
  }
}
```

**Error Codes:**
- `400 Bad Request` (GW-GRP-003) - Role not in owner, admin, member
- `404 Not Found` (GW-GRP-004) - User not found
- `422 Unprocessable Entity` (GW-GRP-005) - Duplicate membership or validation error

---

### DELETE /groups/:id/members/:user_id - Remove Member

Soft delete of the membership. The group owner cannot be removed.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/groups/{group_id}/members/{user_unique_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{ "message": "Member has been successfully removed from the group" }
```

**Error Codes:**
- `403 Forbidden` (GW-GRP-006) - Cannot remove the group owner
- `404 Not Found` (GW-GRP-007) - Member not in group

---

### POST /groups/:id/invitations - Invite by Email

Invites someone, who may not have an account yet, to join the group. Invitations expire after 7 days.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/groups/{group_id}/invitations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "email": "kid@example.com", "role": "member" }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Invitee email (lowercased) |
| `role` | string | No | `admin` or `member` (default `member`; `owner` is not allowed) |

**Response 201:**
```json
{ "data": { "id": "invitation-uuid", "email": "kid@example.com", "role": "member", "expires_at": "2026-11-02T10:30:00Z" } }
```

The response does not include the acceptance token. How the invitee receives it (email or otherwise) is not documented; ask the Auth API team before building a flow that depends on it.

**Error Codes:**
- `422` GW-INV-001 - Email missing
- `400` GW-INV-002 - Invalid role

---

### GET /groups/:id/invitations - List Pending Invitations

Returns the group's pending, unexpired invitations (`id`, `email`, `role`, `expires_at`, `invited_by`).

```bash
curl -X GET "$BLOCKS_API_URL/groups/{group_id}/invitations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

---

### DELETE /groups/:id/invitations/:invitation_id - Revoke Invitation

```bash
curl -X DELETE "$BLOCKS_API_URL/groups/{group_id}/invitations/{invitation_id}" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204** No Content. **Error:** `404` GW-INV-003 - Invitation not found.

---

### POST /groups/invitations/:token/accept - Accept Invitation

The invited user accepts with their own token. Their account email must match the invitation email (case-insensitive).

```bash
curl -X POST "$BLOCKS_API_URL/groups/invitations/{token}/accept" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:** the group, with its members included.

**Error Codes:**
- `404` GW-INV-004 - Invalid token
- `422` GW-INV-005 - Invitation expired
- `422` GW-INV-006 - Already accepted or revoked
- `403` GW-INV-008 - The user's email does not match the invitation
- `422` GW-INV-007 - Accept failed

---

### GET /users/:user_id/groups - User's Groups

All groups a user belongs to. Use it when a token has no group claims.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/{user_unique_id}/groups" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

Requires a token for that user, or `groups:admin`.

---

## JWT Group Claims

When a user belongs to at least one active group, tokens issued for them include:

| Claim | Type | Description |
|-------|------|-------------|
| `group_id` | string (UUID) | Primary group (the user's oldest active membership) |
| `group_owner_id` | string (UUID) | Owner of the primary group, the purchaser for billing and support |
| `group_role` | string | The user's role in the primary group: `owner`, `admin` or `member` |
| `group_ids` | array | Every group the user belongs to |

The claims are in the legacy login token, in OAuth-mode access tokens (sign-in with `X-OAuth-Mode: true`) and in tokens minted by a refresh, so a refreshed token keeps them (this was fixed in Auth v4.49.2; older tokens dropped them on refresh). They are absent when the user has no active group. A token issued before the user joined a group does not have them: refresh or sign in again, or look the groups up with `GET /users/:user_id/groups`.

Decode a token locally to inspect the claims:
```bash
echo "$BLOCKS_AUTH_TOKEN" | cut -d. -f2 | tr '_-' '/+' | base64 -d 2>/dev/null | jq '{group_id, group_owner_id, group_role, group_ids}'
```

## Checking a Group Entitlement

Group membership alone does not prove the plan is paid. Read `group_id` from the token, then check the entitlement for that group in the system that holds the subscription (for example the Sales block), and grant access only if it is active.

1. Read `group_id` (and `group_ids` for multi-group users) from the token. If missing, call `GET /users/:user_id/groups`.
2. Check the entitlement keyed by that group ID.
3. Use `group_owner_id` to identify the purchaser.

## Data Model

**Group:** `unique_id`, `name`, `description`, `owner_user_id`, `status` (`active`, `suspended`, `deleted`), `enabled`.

**GroupMember:** `user_unique_id`, `role` (`owner`, `admin`, `member`), `status`, `enabled`. A user has at most one membership per group.

## Notes

- **Owner protection**: the owner cannot be removed or replaced. Ownership transfer is not available yet.
- **Soft delete**: removed members and deleted groups are flagged, never erased.
- **Invitations**: invite by email with `POST /groups/:id/invitations`, or add an existing user directly with `POST /groups/:id/members`. These are separate from user-signup invitations (`23blocks-auth-invitations-api`).
- **Routes**: every group route also exists under `/:company_url_id/groups`.
- **Fields**: the group's `enabled` flag is never returned.
- **Not the same as other grouping features**: access-control teams are `23blocks-auth-teams-api`, organizations are `23blocks-auth-organizations-api`, and chat groups are `23blocks-conversations-groups-api`.
