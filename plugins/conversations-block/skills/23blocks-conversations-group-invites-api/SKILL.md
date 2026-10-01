---
name: 23blocks-conversations-group-invites-api
description: "Conversations Block group invite codes: expiry, usage limits, QR, revoke, join by code. Use for shareable group invites."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Group Invites API

Create and manage shareable invite codes for joining groups. Supports expiration dates, usage limits, QR code generation, revocation, and rate-limited join attempts.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://realtime.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/groups/:group_unique_id/invites` | List active invites for a group |
| POST | `/groups/:group_unique_id/invites` | Create a new invite code |
| DELETE | `/groups/:group_unique_id/invites/:code` | Revoke an invite code |
| GET | `/groups/:group_unique_id/invites/:code/qr` | Get QR code image for invite |
| POST | `/groups/join/:code` | Join a group via invite code |

---

## Data Model

### GroupInvite

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the invite |
| code | string | URL-safe invite code (22 chars, 132 bits entropy) |
| group_unique_id | string | Associated group ID |
| name | string | Optional invite name/label |
| created_by | string | User who created the invite |
| status | string | Status: `active`, `revoked`, `expired` |
| enabled | string | `true` or `false` |
| max_uses | integer | Maximum uses allowed (null for unlimited) |
| use_count | integer | Number of times the code has been used |
| expires_at | datetime | Expiration timestamp (null for no expiry) |
| created_at | datetime | Invite creation timestamp |
| updated_at | datetime | Last update timestamp |

### Invite Usability

An invite is usable when all conditions are met:
- `status == 'active'`
- `enabled == 'true'`
- Not expired (`expires_at` is null or in the future)
- Not at max uses (`use_count < max_uses` or `max_uses` is null)

---

## Rate Limiting

Join attempts are rate-limited to **10 attempts per hour per IP address**. Exceeding this returns `429 Too Many Requests`.

---

## Error Response Format

```json
{
  "errors": [
    {
      "status": "401",
      "title": "Unauthorized",
      "detail": "Invalid or missing authentication token"
    }
  ]
}
```

| Status | Meaning |
|--------|---------|
| `401` | Unauthorized — invalid or missing token |
| `403` | Forbidden — insufficient permissions |
| `404` | Not Found — invalid invite code |
| `409` | Conflict — user already a member |
| `410` | Gone — invite expired, revoked, or max uses reached |
| `422` | Unprocessable Entity — validation error |
| `429` | Too Many Requests — rate limit exceeded |
