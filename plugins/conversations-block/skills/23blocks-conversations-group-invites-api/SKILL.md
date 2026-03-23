---
name: 23blocks-conversations-group-invites-api
description: Create and manage shareable group invite codes with QR generation, expiration, usage limits, and join-via-code. Use when generating invite links, managing invite lifecycle, or enabling users to join groups.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Group Invites API

Create and manage shareable invite codes for joining groups. Supports expiration dates, usage limits, QR code generation, revocation, and rate-limited join attempts.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Conversations API base URL | `https://realtime.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://realtime.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://realtime.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


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
