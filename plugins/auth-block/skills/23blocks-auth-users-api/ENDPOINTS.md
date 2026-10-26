# Users API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

## Contents

- GET `/users`: List Users
- GET `/users/:unique_id`: Get User
- POST `/users`: Create User
- PUT `/users/:unique_id`: Update User
- DELETE `/users/:unique_id`: Delete User
- GET `/users/me`: Get Current Profile
- PUT `/users/me`: Update Current Profile
- PUT `/users/:unique_id/change_password`: Change Password
- POST `/users/reset_password`: Request Password Reset
- POST `/users/verify_email`: Verify Email
- PUT `/users/:unique_id/activate`: Activate User
- PUT `/users/:unique_id/deactivate`: Deactivate User
- POST `/users/search`: Search Users
- DELETE `/me`: Delete My Account
- POST `/users/recover`: Request Account Recovery
- PUT `/users/recover`: Complete Account Recovery

---

## Endpoints

### GET /users - List Users

Lists all users with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "user-uuid-123",
      "type": "user",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "user@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "status": "active",
        "email_verified": true,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### GET /users/:unique_id - Get User

Retrieves a single user by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "user@example.com",
      "first_name": "John",
      "last_name": "Doe",
      "status": "active",
      "email_verified": true,
      "last_login_at": "2025-01-12T08:00:00Z",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "roles": {
        "data": [
          { "id": "role-uuid", "type": "role" }
        ]
      },
      "organization": {
        "data": { "id": "org-uuid", "type": "organization" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### POST /users - Create User

Creates a new user account.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "password": "secure_password_123",
      "first_name": "Jane",
      "last_name": "Smith"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email |
| `password` | string | Yes | User password |
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |

**Response 201:**
```json
{
  "data": {
    "id": "new-user-uuid",
    "type": "user",
    "attributes": {
      "unique_id": "new-user-uuid",
      "email": "newuser@example.com",
      "first_name": "Jane",
      "last_name": "Smith",
      "status": "active",
      "email_verified": false,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Email already taken
- `422 Unprocessable Entity` - Validation errors

---

### PUT /users/:unique_id - Update User

Updates an existing user account.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "first_name": "Jonathan",
      "last_name": "Doe"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "first_name": "Jonathan",
      "last_name": "Doe",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### DELETE /users/:unique_id - Delete User

Soft-deletes a user account.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - User not found

---

### GET /users/me - Get Current Profile

Retrieves the current authenticated user's profile.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/me" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "current-user-uuid",
    "type": "user",
    "attributes": {
      "unique_id": "current-user-uuid",
      "email": "me@example.com",
      "first_name": "John",
      "last_name": "Doe",
      "status": "active",
      "email_verified": true,
      "last_login_at": "2025-01-12T08:00:00Z",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### PUT /users/me - Update Current Profile

Updates the current authenticated user's profile.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/me" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "first_name": "John",
      "last_name": "Updated"
    }
  }'
```

**Response 200:** Same format as GET /users/me

---

### PUT /users/:unique_id/change_password - Change Password

Changes a user's password.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/change_password" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "current_password": "old_password",
      "new_password": "new_secure_password",
      "new_password_confirmation": "new_secure_password"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `current_password` | string | Yes | Current password |
| `new_password` | string | Yes | New password |
| `new_password_confirmation` | string | Yes | Password confirmation |

**Response 200:**
```json
{
  "message": "Password changed successfully"
}
```

**Errors:**
- `401 Unauthorized` - Current password incorrect
- `422 Unprocessable Entity` - Password does not meet requirements

---

### POST /users/reset_password - Request Password Reset

Sends a password reset email.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/reset_password" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "user@example.com"
    }
  }'
```

**Response 200:**
```json
{
  "message": "Password reset instructions sent"
}
```

---

### POST /users/verify_email - Verify Email

Verifies a user's email address using the verification token.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/verify_email" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "token": "email-verification-token"
  }'
```

**Response 200:**
```json
{
  "message": "Email verified successfully"
}
```

**Errors:**
- `422 Unprocessable Entity` - Invalid or expired token

---

### PUT /users/:unique_id/activate - Activate User

Activates a deactivated user account.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/activate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### PUT /users/:unique_id/deactivate - Deactivate User

Deactivates a user account.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/deactivate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "status": "inactive",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### POST /users/search - Search Users

Searches users with filters.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "email": "example.com",
      "status": "active"
    },
    "page": 1,
    "records": 20
  }'
```

**Response 200:** Same format as GET /users

---

### DELETE /me - Delete My Account

Self-service account deletion (Auth API v4.49.0). The signed-in user deletes their own account; no admin scope is needed. It is a soft delete: the user row stays with `status: deleted` and `deleted_at` set, and `unique_id` is kept so other blocks' records still resolve.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/me" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "password": "current-password",
    "anonymize_after_grace_period": true
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `password` | string | Yes | The user's current password |
| `anonymize_after_grace_period` | boolean | No | If true, a job anonymizes email, name, profile PII and avatar after the recovery window |

**Response 204** No Content.

**What happens:**
- Refresh tokens are revoked in the same transaction, so the user is signed out everywhere.
- Deleted, suspended and banned users can no longer sign in (password or passwordless).
- The recovery window is 6 hours (Devise `reset_password_within`). With anonymization, the email becomes `anonymized_<user_id>_<timestamp>@deleted.local`.
- Other blocks keep their own copies of the user. Auth does not cascade the delete: remove or cancel subscriptions, Jarvis identity, files and forms in the other blocks first, and delete in Auth last.

**Errors:**
- `404` GW-DEL-001 - User not found
- `422` GW-DEL-002 - Password missing
- `401` GW-DEL-003 - Wrong password
- `422` GW-DEL-004 - Already deleted

---

### POST /users/recover - Request Account Recovery

Emails a recovery link for a deleted account that is still inside the recovery window. Always answers `200` with the same message whether or not the email exists or the account is deleted, so it cannot be used to probe for accounts.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/recover" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "email": "user@example.com", "recovery_url": "https://app.example.com/recover" }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Account email |
| `recovery_url` | string | Yes | Page in your app that receives the recovery token |

**Errors:** `422` GW-REC-001 (email missing), `422` GW-REC-002 (recovery URL missing).

---

### PUT /users/recover - Complete Account Recovery

Reactivates the deleted account and sets a new password.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/recover" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "recovery_token": "token-from-email",
    "password": "new-password",
    "password_confirmation": "new-password"
  }'
```

**Response 200:** the user, now `status: active`.

**Errors:** `422` GW-REC-010 (token missing), GW-REC-011 (password missing), GW-REC-012 (passwords differ), GW-REC-013 (invalid token), GW-REC-014 (expired token), GW-REC-015 (account is not deleted), GW-REC-016 (save failed).

An account that was already anonymized cannot be meaningfully recovered.
