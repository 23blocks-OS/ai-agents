# Group Invites API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Endpoints

### List Group Invites

Retrieve all active invite codes for a group. Results are ordered by creation date (newest first).

**Endpoint:** `GET /groups/:group_unique_id/invites`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_unique_id | string | Yes | The group's unique identifier |

**Permission:** `groups:read`

**cURL Example:**

```bash
curl -s -X GET "$BLOCKS_API_URL/groups/grp_001/invites" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "inv_001",
      "type": "group_invites",
      "attributes": {
        "unique_id": "inv_001",
        "code": "Xk9Lm3Np7Qr2St4Uv6Wx-_",
        "group_unique_id": "grp_001",
        "name": "Team onboarding link",
        "created_by": "usr_abc123",
        "status": "active",
        "enabled": "true",
        "max_uses": 50,
        "use_count": 12,
        "expires_at": "2025-03-01T00:00:00Z",
        "created_at": "2025-01-15T10:00:00Z",
        "updated_at": "2025-01-15T10:00:00Z"
      }
    }
  ]
}
```

---

### Create Group Invite

Generate a new invite code for a group. The code is auto-generated as a URL-safe base64 string.

**Endpoint:** `POST /groups/:group_unique_id/invites`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_unique_id | string | Yes | The group's unique identifier |

**Permission:** `groups:write`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | No | Label for the invite (e.g., "Team onboarding") |
| expires_at | string | No | ISO 8601 expiration timestamp |
| expires_in_hours | integer | No | Hours until expiration (alternative to `expires_at`) |
| max_uses | integer | No | Maximum number of uses (null for unlimited) |

> **Note:** Provide either `expires_at` or `expires_in_hours`, not both.

**cURL Example — With expiration date:**

```bash
curl -s -X POST "$BLOCKS_API_URL/groups/grp_001/invites" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Team onboarding link",
    "max_uses": 50,
    "expires_at": "2025-03-01T00:00:00Z"
  }'
```

**cURL Example — With expiration in hours:**

```bash
curl -s -X POST "$BLOCKS_API_URL/groups/grp_001/invites" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "24-hour invite",
    "expires_in_hours": 24,
    "max_uses": 10
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "inv_new001",
    "type": "group_invites",
    "attributes": {
      "unique_id": "inv_new001",
      "code": "Ab3Cd5Ef7Gh9Ij1Kl3Mn-_",
      "group_unique_id": "grp_001",
      "name": "Team onboarding link",
      "created_by": "usr_abc123",
      "status": "active",
      "enabled": "true",
      "max_uses": 50,
      "use_count": 0,
      "expires_at": "2025-03-01T00:00:00Z",
      "created_at": "2025-01-15T14:30:00Z",
      "updated_at": "2025-01-15T14:30:00Z"
    }
  }
}
```

---

### Revoke Invite

Revoke an invite code so it can no longer be used. Sets the invite status to `revoked`.

**Endpoint:** `DELETE /groups/:group_unique_id/invites/:code`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_unique_id | string | Yes | The group's unique identifier |
| code | string | Yes | The invite code to revoke |

**Permission:** `groups:write`

**cURL Example:**

```bash
curl -s -X DELETE "$BLOCKS_API_URL/groups/grp_001/invites/Xk9Lm3Np7Qr2St4Uv6Wx-_" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "inv_001",
    "type": "group_invites",
    "attributes": {
      "unique_id": "inv_001",
      "code": "Xk9Lm3Np7Qr2St4Uv6Wx-_",
      "status": "revoked",
      "updated_at": "2025-01-15T15:00:00Z"
    }
  }
}
```

---

### Get Invite QR Code

Generate a QR code image for an invite code. The QR encodes a join URL.

**Endpoint:** `GET /groups/:group_unique_id/invites/:code/qr`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_unique_id | string | Yes | The group's unique identifier |
| code | string | Yes | The invite code |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| size | integer | No | QR code image size in pixels (default: 300) |

**Permission:** `groups:read`

**cURL Example:**

```bash
curl -s -X GET "$BLOCKS_API_URL/groups/grp_001/invites/Xk9Lm3Np7Qr2St4Uv6Wx-_/qr?size=512" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  --output invite_qr.png
```

**Response (200 OK):**

Returns a PNG image binary with `Content-Type: image/png`.

> **Note:** Requires the `rqrcode` gem on the server. Returns `501 Not Implemented` if unavailable.

---

### Join Group via Invite Code

Join a group using an invite code. The authenticated user is added as a member. Optionally provide user profile details.

**Endpoint:** `POST /groups/join/:code`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| code | string | Yes | The invite code |

**Permission:** `groups:read` (any authenticated user can attempt to join)

**Request Body (optional):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| user.email | string | No | User's email |
| user.first_name | string | No | User's first name |
| user.last_name | string | No | User's last name |
| user.phone | string | No | User's phone |
| user.avatar_url | string | No | User's avatar URL |
| user.role_name | string | No | Role name |
| user.role_unique_id | string | No | Role unique ID |
| user.preferred_language | string | No | Preferred language |
| user.time_zone | string | No | Time zone |
| user.name | string | No | Display name |
| user.source | string | No | Source identifier |
| user.source_id | string | No | Source ID |
| user.source_alias | string | No | Source alias |
| user.source_type | string | No | Source type |

**cURL Example:**

```bash
curl -s -X POST "$BLOCKS_API_URL/groups/join/Ab3Cd5Ef7Gh9Ij1Kl3Mn-_" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "name": "Jane Smith",
      "email": "jane@example.com",
      "role_name": "member"
    }
  }'
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "grp_001",
    "type": "groups",
    "attributes": {
      "unique_id": "grp_001",
      "name": "Engineering Team",
      "status": "active"
    },
    "relationships": {
      "group_conversations": {
        "data": [
          { "id": "gc_001", "type": "group_conversations" }
        ]
      }
    }
  }
}
```

**Error Responses:**

| Status | Error Code | Meaning |
|--------|------------|---------|
| 404 | `invalid_code` | Invite code not found |
| 409 | `already_member` | User is already a group member |
| 410 | `expired` | Invite has expired |
| 410 | `revoked` | Invite was revoked |
| 410 | `max_uses_reached` | Invite has reached its usage limit |
| 429 | `rate-limit` | Too many join attempts (max 10/hour per IP) |
