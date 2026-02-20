---
name: conversations-groups-api
description: Create and manage groups, members, invites, and group conversations in the 23blocks Conversations Block
version: 1.0.0
tags:
  - conversations
  - groups
  - members
  - invites
  - qr-codes
env:
  - BLOCKS_API_URL
  - BLOCKS_AUTH_TOKEN
  - BLOCKS_API_KEY
---

# Groups API

Create and manage groups of users. Supports member management, group conversations, invite codes with QR generation, and context-based grouping.

## Authentication

All requests require the following headers:

```
Authorization: Bearer $BLOCKS_AUTH_TOKEN
AppId: $BLOCKS_API_KEY
```

## Base URL

```
https://conversations.api.us.23blocks.com
```

---

## Endpoints

### List User Groups

Retrieve all groups that a user belongs to.

**Endpoint:** `GET /users/:unique_id/groups`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The user's unique identifier |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number for pagination |
| per_page | integer | No | Results per page (default: 25) |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/groups?page=1&per_page=25" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "grp_001",
      "type": "groups",
      "attributes": {
        "unique_id": "grp_001",
        "name": "Engineering Team",
        "description": "Main engineering discussion group",
        "members": ["usr_abc123", "usr_xyz789", "usr_lmn456"],
        "owner_id": "usr_abc123",
        "context_id": "ctx_eng001",
        "status": "active",
        "member_count": 3,
        "created_at": "2025-01-10T08:00:00Z",
        "updated_at": "2025-01-14T16:00:00Z"
      }
    }
  ],
  "meta": {
    "total": 4,
    "page": 1,
    "per_page": 25
  }
}
```

---

### Get Group Members

Retrieve all members of a specific group.

**Endpoint:** `GET /groups/:group_unique_id/members`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_unique_id | string | Yes | The group's unique identifier |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number for pagination |
| per_page | integer | No | Results per page (default: 25) |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/groups/grp_001/members?page=1&per_page=25" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "usr_abc123",
      "type": "group_members",
      "attributes": {
        "user_unique_id": "usr_abc123",
        "display_name": "John Doe",
        "role": "owner",
        "joined_at": "2025-01-10T08:00:00Z",
        "status": "online"
      }
    },
    {
      "id": "usr_xyz789",
      "type": "group_members",
      "attributes": {
        "user_unique_id": "usr_xyz789",
        "display_name": "Jane Smith",
        "role": "member",
        "joined_at": "2025-01-10T09:00:00Z",
        "status": "away"
      }
    }
  ],
  "meta": {
    "total": 3,
    "page": 1,
    "per_page": 25
  }
}
```

---

### Create Group

Create a new group.

**Endpoint:** `POST /groups/`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Group name |
| description | string | No | Group description |
| owner_id | string | Yes | User unique ID of the group owner |
| members | array | No | Initial member user unique IDs |
| context_id | string | No | Context to associate the group with |
| metadata | object | No | Additional metadata |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/groups/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Engineering Team",
    "description": "Main engineering discussion group",
    "owner_id": "usr_abc123",
    "members": ["usr_abc123", "usr_xyz789"],
    "metadata": {
      "department": "engineering"
    }
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "grp_new001",
    "type": "groups",
    "attributes": {
      "unique_id": "grp_new001",
      "name": "Engineering Team",
      "description": "Main engineering discussion group",
      "members": ["usr_abc123", "usr_xyz789"],
      "owner_id": "usr_abc123",
      "context_id": null,
      "status": "active",
      "member_count": 2,
      "metadata": {
        "department": "engineering"
      },
      "created_at": "2025-01-15T11:00:00Z",
      "updated_at": "2025-01-15T11:00:00Z"
    }
  }
}
```

---

### Update Group

Update an existing group's details.

**Endpoint:** `PUT /groups/:unique_id/`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The group's unique identifier |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | No | Updated group name |
| description | string | No | Updated description |
| metadata | object | No | Updated metadata |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/groups/grp_001/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Engineering Core Team",
    "description": "Core engineering leads"
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
      "name": "Engineering Core Team",
      "description": "Core engineering leads",
      "updated_at": "2025-01-15T12:00:00Z"
    }
  }
}
```

---

### Get Group Conversation

Retrieve a specific conversation within a group.

**Endpoint:** `GET /groups/:group_unique_id/conversations/:conversation_unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_unique_id | string | Yes | The group's unique identifier |
| conversation_unique_id | string | Yes | The conversation's unique identifier |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/groups/grp_001/conversations/conv_grp789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "conv_grp789",
    "type": "conversations",
    "attributes": {
      "unique_id": "conv_grp789",
      "group_id": "grp_001",
      "participants": ["usr_abc123", "usr_xyz789", "usr_lmn456"],
      "last_message": {
        "content": "Sprint planning tomorrow",
        "sender_id": "usr_abc123",
        "sent_at": "2025-01-15T10:00:00Z"
      },
      "unread_count": 2,
      "status": "active",
      "created_at": "2025-01-10T08:00:00Z",
      "updated_at": "2025-01-15T10:00:00Z"
    }
  }
}
```

---

### Add Member to Group

Add a user to an existing group.

**Endpoint:** `PUT /groups/:unique_id/users/:user_unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The group's unique identifier |
| user_unique_id | string | Yes | The user's unique identifier to add |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| role | string | No | Member role: `member` (default), `admin`, `moderator` |

**cURL Example:**

```bash
curl -s -X PUT "https://conversations.api.us.23blocks.com/groups/grp_001/users/usr_new001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "role": "member"
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
      "member_count": 4,
      "members": ["usr_abc123", "usr_xyz789", "usr_lmn456", "usr_new001"],
      "updated_at": "2025-01-15T12:30:00Z"
    }
  }
}
```

---

### Remove Member from Group

Remove a user from a group.

**Endpoint:** `DELETE /groups/:unique_id/users/:user_unique_id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The group's unique identifier |
| user_unique_id | string | Yes | The user's unique identifier to remove |

**cURL Example:**

```bash
curl -s -X DELETE "https://conversations.api.us.23blocks.com/groups/grp_001/users/usr_new001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "grp_001",
    "type": "groups",
    "attributes": {
      "unique_id": "grp_001",
      "member_count": 3,
      "members": ["usr_abc123", "usr_xyz789", "usr_lmn456"],
      "updated_at": "2025-01-15T13:00:00Z"
    }
  }
}
```

---

### Create Group with Context

Create a group associated with a specific context, along with an initial conversation.

**Endpoint:** `POST /groups/conversations/contexts`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Group name |
| description | string | No | Group description |
| owner_id | string | Yes | User unique ID of the group owner |
| members | array | No | Initial member user unique IDs |
| context_id | string | Yes | Context to associate the group with |
| conversation_name | string | No | Name for the initial conversation |
| metadata | object | No | Additional metadata |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/groups/conversations/contexts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Project Alpha Team",
    "description": "Team for Project Alpha",
    "owner_id": "usr_abc123",
    "members": ["usr_abc123", "usr_xyz789"],
    "context_id": "ctx_proj_alpha",
    "conversation_name": "General"
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "grp_ctx001",
    "type": "groups",
    "attributes": {
      "unique_id": "grp_ctx001",
      "name": "Project Alpha Team",
      "description": "Team for Project Alpha",
      "members": ["usr_abc123", "usr_xyz789"],
      "owner_id": "usr_abc123",
      "context_id": "ctx_proj_alpha",
      "status": "active",
      "member_count": 2,
      "conversation": {
        "unique_id": "conv_ctx001",
        "name": "General"
      },
      "created_at": "2025-01-15T14:00:00Z",
      "updated_at": "2025-01-15T14:00:00Z"
    }
  }
}
```

---

### List Context Groups

Retrieve all groups associated with a specific context for a user.

**Endpoint:** `GET /users/:unique_id/context/:context_unique_id/groups`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| unique_id | string | Yes | The user's unique identifier |
| context_unique_id | string | Yes | The context's unique identifier |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/users/usr_abc123/context/ctx_proj_alpha/groups" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "grp_ctx001",
      "type": "groups",
      "attributes": {
        "unique_id": "grp_ctx001",
        "name": "Project Alpha Team",
        "context_id": "ctx_proj_alpha",
        "member_count": 2,
        "status": "active"
      }
    }
  ],
  "meta": {
    "total": 1,
    "page": 1,
    "per_page": 25
  }
}
```

---

## Invites

### List Group Invites

Retrieve all active invite codes for a group.

**Endpoint:** `GET /groups/:group_unique_id/invites`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_unique_id | string | Yes | The group's unique identifier |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/groups/grp_001/invites" \
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
      "type": "invites",
      "attributes": {
        "code": "ABC123XY",
        "group_unique_id": "grp_001",
        "created_by": "usr_abc123",
        "max_uses": 10,
        "use_count": 3,
        "expires_at": "2025-02-15T00:00:00Z",
        "created_at": "2025-01-15T10:00:00Z"
      }
    }
  ]
}
```

---

### Create Group Invite

Generate a new invite code for a group.

**Endpoint:** `POST /groups/:group_unique_id/invites`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_unique_id | string | Yes | The group's unique identifier |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| max_uses | integer | No | Maximum number of times the code can be used |
| expires_at | string | No | ISO 8601 expiration time |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/groups/grp_001/invites" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "max_uses": 50,
    "expires_at": "2025-03-01T00:00:00Z"
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "inv_new001",
    "type": "invites",
    "attributes": {
      "code": "XYZ789AB",
      "group_unique_id": "grp_001",
      "created_by": "usr_abc123",
      "max_uses": 50,
      "use_count": 0,
      "expires_at": "2025-03-01T00:00:00Z",
      "created_at": "2025-01-15T14:30:00Z"
    }
  }
}
```

---

### Delete Group Invite

Delete/revoke an invite code.

**Endpoint:** `DELETE /groups/:group_unique_id/invites/:code`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_unique_id | string | Yes | The group's unique identifier |
| code | string | Yes | The invite code to delete |

**cURL Example:**

```bash
curl -s -X DELETE "https://conversations.api.us.23blocks.com/groups/grp_001/invites/ABC123XY" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "inv_001",
    "type": "invites",
    "attributes": {
      "code": "ABC123XY",
      "deleted": true,
      "deleted_at": "2025-01-15T15:00:00Z"
    }
  }
}
```

---

### Get Invite QR Code

Generate a QR code image for an invite code.

**Endpoint:** `GET /groups/:group_unique_id/invites/:code/qr`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_unique_id | string | Yes | The group's unique identifier |
| code | string | Yes | The invite code |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| size | integer | No | QR code size in pixels (default: 256) |
| format | string | No | Image format: `png` (default), `svg` |

**cURL Example:**

```bash
curl -s -X GET "https://conversations.api.us.23blocks.com/groups/grp_001/invites/ABC123XY/qr?size=512&format=png" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  --output invite_qr.png
```

**Response (200 OK):**

Returns the QR code image binary with `Content-Type: image/png` (or `image/svg+xml` for SVG format).

---

### Join Group via Invite Code

Join a group using an invite code.

**Endpoint:** `POST /groups/join/:code`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| code | string | Yes | The invite code |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| user_unique_id | string | Yes | The user's unique identifier |

**cURL Example:**

```bash
curl -s -X POST "https://conversations.api.us.23blocks.com/groups/join/XYZ789AB" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "usr_new002"
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
      "member_count": 4,
      "joined": true,
      "joined_at": "2025-01-15T15:30:00Z"
    }
  }
}
```

---

## Data Model

### Group

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the group |
| name | string | Group name |
| description | string | Group description |
| members | array | List of member user unique IDs |
| owner_id | string | User unique ID of the group owner |
| context_id | string | Associated context ID |
| status | string | Group status: `active`, `archived`, `deleted` |
| member_count | integer | Current number of members |
| metadata | object | Arbitrary key-value metadata |
| created_at | datetime | Group creation timestamp |
| updated_at | datetime | Last update timestamp |

### Invite

| Field | Type | Description |
|-------|------|-------------|
| code | string | Unique invite code |
| group_unique_id | string | Associated group ID |
| created_by | string | User who created the invite |
| max_uses | integer | Maximum uses allowed (null for unlimited) |
| use_count | integer | Number of times the code has been used |
| expires_at | datetime | Expiration timestamp (null for never) |
| created_at | datetime | Invite creation timestamp |

---

## Error Responses

### 401 Unauthorized

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

### 403 Forbidden

```json
{
  "errors": [
    {
      "status": "403",
      "title": "Forbidden",
      "detail": "Only group owners and admins can add members"
    }
  ]
}
```

### 404 Not Found

```json
{
  "errors": [
    {
      "status": "404",
      "title": "Not Found",
      "detail": "Group with unique_id 'grp_invalid' not found"
    }
  ]
}
```

### 410 Gone

```json
{
  "errors": [
    {
      "status": "410",
      "title": "Gone",
      "detail": "Invite code 'ABC123XY' has expired or been revoked"
    }
  ]
}
```

### 422 Unprocessable Entity

```json
{
  "errors": [
    {
      "status": "422",
      "title": "Unprocessable Entity",
      "detail": "User is already a member of this group"
    }
  ]
}
```
