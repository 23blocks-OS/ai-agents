# File Access Control API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Access Control Endpoints

### GET /users/:unique_id/files/:unique_file_id/access - Get File Access

Retrieves the access list for a file.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "access-id-1",
      "type": "user_file_access",
      "attributes": {
        "unique_id": "access-id-1",
        "user_unique_id": "user-123",
        "access_type": "owner",
        "starts_at": "2025-01-10T10:30:00Z",
        "expires_at": null,
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "access-id-2",
      "type": "user_file_access",
      "attributes": {
        "unique_id": "access-id-2",
        "user_unique_id": "user-456",
        "access_type": "read",
        "starts_at": "2025-01-12T09:00:00Z",
        "expires_at": "2025-06-12T09:00:00Z",
        "created_at": "2025-01-12T09:00:00Z"
      }
    }
  ]
}
```

---

### POST /users/:unique_id/files/:unique_file_id/access/grant - Grant Access

Grants access to a file for another user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/grant" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access": {
      "user_unique_id": "target-user-id",
      "access_type": "read",
      "expires_at": "2025-12-31T23:59:59Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | string | Yes | User to grant access to |
| `access_type` | enum | Yes | owner, write, read |
| `expires_at` | timestamp | No | When access expires |

**Response 201:**
```json
{
  "data": {
    "id": "new-access-id",
    "type": "user_file_access",
    "attributes": {
      "unique_id": "new-access-id",
      "user_unique_id": "target-user-id",
      "access_type": "read",
      "starts_at": "2025-01-10T10:30:00Z",
      "expires_at": "2025-12-31T23:59:59Z"
    }
  }
}
```

---

### DELETE /users/:unique_id/files/:unique_file_id/access/:access_unique_id/revoke - Revoke Access

Revokes a specific access grant.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/$ACCESS_ID/revoke" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### POST /users/:unique_id/files/:unique_file_id/access/make_public - Make File Public

Changes file access level to public.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/make_public" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "file-id",
    "type": "user_file",
    "attributes": {
      "access_level": "public"
    }
  }
}
```

---

### POST /users/:unique_id/files/:unique_file_id/access/make_private - Make File Private

Changes file access level to private.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/make_private" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "file-id",
    "type": "user_file",
    "attributes": {
      "access_level": "private"
    }
  }
}
```

---

## Bulk Operations

### POST /users/:unique_id/files/access/grant - Bulk Grant Access

Grants a user access to multiple files at once.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/access/grant" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access": {
      "user_unique_id": "target-user-id",
      "access_type": "read",
      "file_unique_ids": ["file-1", "file-2", "file-3"]
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | string | Yes | User to grant access to |
| `access_type` | enum | Yes | owner, write, read |
| `file_unique_ids` | array | Yes | List of file IDs |

**Response 200:**
```json
{
  "data": {
    "granted": 3,
    "failed": 0,
    "results": [
      {"file_id": "file-1", "status": "granted"},
      {"file_id": "file-2", "status": "granted"},
      {"file_id": "file-3", "status": "granted"}
    ]
  }
}
```

---

### POST /users/:unique_id/files/access/revoke - Bulk Revoke Access

Revokes a user's access from multiple files.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/access/revoke" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access": {
      "user_unique_id": "target-user-id",
      "file_unique_ids": ["file-1", "file-2", "file-3"]
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "revoked": 3,
    "failed": 0
  }
}
```

---

### POST /users/:unique_id/files/access/grant-to-users - Grant Access to Multiple Users

Grants multiple users access to all of a user's files.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/access/grant-to-users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access": {
      "user_unique_ids": ["user-1", "user-2", "user-3"],
      "access_type": "read"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_ids` | array | Yes | Users to grant access to |
| `access_type` | enum | Yes | owner, write, read |

**Response 200:**
```json
{
  "data": {
    "users_granted": 3,
    "files_affected": 25
  }
}
```

---

### POST /users/:unique_id/files/access/revoke-from-users - Revoke Access from Multiple Users

Revokes multiple users' access from all of a user's files.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/access/revoke-from-users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access": {
      "user_unique_ids": ["user-1", "user-2", "user-3"]
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "users_revoked": 3,
    "files_affected": 25
  }
}
```

---

## Access Request Endpoints

### POST /users/:unique_id/files/:unique_file_id/requests/access - Request Access

Submits an access request for a file.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/requests/access" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "request": {
      "access_type": "read",
      "reason": "Need access for project review"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `access_type` | enum | Yes | owner, write, read |
| `reason` | string | No | Reason for request |

**Response 201:**
```json
{
  "data": {
    "id": "request-id",
    "type": "user_file_access_request",
    "attributes": {
      "unique_id": "request-id",
      "access_type": "read",
      "status": "pending",
      "requested_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### GET /users/:unique_id/files/:unique_file_id/access/requests - List Access Requests

Lists pending access requests for a file.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/requests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "request-id",
      "type": "user_file_access_request",
      "attributes": {
        "unique_id": "request-id",
        "user_unique_id": "requesting-user-id",
        "access_type": "read",
        "status": "pending",
        "requested_by_user_name": "John Doe",
        "requested_by_email": "john@example.com",
        "requested_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### PUT /users/:unique_id/files/:unique_file_id/access/requests/:request_unique_id/approve - Approve Request

Approves an access request and grants access.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/requests/$REQUEST_ID/approve" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "request-id",
    "type": "user_file_access_request",
    "attributes": {
      "status": "approved",
      "approved_at": "2025-01-10T14:00:00Z",
      "approved_by_user_name": "File Owner"
    }
  }
}
```

---

### DELETE /users/:unique_id/files/:unique_file_id/access/requests/:request_unique_id/deny - Deny Request

Denies an access request.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/$USER_ID/files/$FILE_ID/access/requests/$REQUEST_ID/deny" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content
