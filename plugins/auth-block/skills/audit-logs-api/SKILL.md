---
name: auth-audit-logs-api
description: Query 23blocks audit logs via REST API. Use when viewing security audit trails, querying specific audit events, or filtering audit logs by user, action, or resource.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Audit Logs API

Complete API reference for 23blocks security audit log querying.

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
curl -X GET "$BLOCKS_API_URL/audit_logs" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /audit_logs - List Audit Logs

Lists audit log entries with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/audit_logs?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
      "id": "log-uuid-123",
      "type": "audit_log",
      "attributes": {
        "unique_id": "log-uuid-123",
        "action": "user.login",
        "actor_id": "user-uuid-123",
        "actor_email": "admin@example.com",
        "resource_type": "session",
        "resource_id": "session-uuid-456",
        "ip_address": "192.168.1.1",
        "user_agent": "Mozilla/5.0...",
        "status": "success",
        "metadata": {
          "method": "password"
        },
        "created_at": "2025-01-12T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 50,
    "totalRecords": 1000
  }
}
```

---

### GET /audit_logs/:unique_id - Get Audit Log Entry

Retrieves a single audit log entry by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/audit_logs/log-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "log-uuid-123",
    "type": "audit_log",
    "attributes": {
      "unique_id": "log-uuid-123",
      "action": "user.login",
      "actor_id": "user-uuid-123",
      "actor_email": "admin@example.com",
      "resource_type": "session",
      "resource_id": "session-uuid-456",
      "ip_address": "192.168.1.1",
      "user_agent": "Mozilla/5.0...",
      "status": "success",
      "metadata": {
        "method": "password",
        "mfa_used": false
      },
      "changes": {},
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Audit log entry not found

---

### POST /audit_logs/query - Query Audit Logs

Queries audit logs with advanced filters.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/audit_logs/query" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "action": "user.login",
      "actor_id": "user-uuid-123",
      "status": "failure",
      "from": "2025-01-01T00:00:00Z",
      "to": "2025-01-31T23:59:59Z"
    },
    "page": 1,
    "records": 50
  }'
```

**Query Filter Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | No | Filter by action type |
| `actor_id` | uuid | No | Filter by actor user ID |
| `actor_email` | string | No | Filter by actor email |
| `resource_type` | string | No | Filter by resource type |
| `resource_id` | uuid | No | Filter by resource ID |
| `status` | string | No | Filter by status (success, failure) |
| `ip_address` | string | No | Filter by IP address |
| `from` | timestamp | No | Start date filter |
| `to` | timestamp | No | End date filter |

**Response 200:**
```json
{
  "data": [
    {
      "id": "log-uuid-789",
      "type": "audit_log",
      "attributes": {
        "unique_id": "log-uuid-789",
        "action": "user.login",
        "actor_id": "user-uuid-123",
        "actor_email": "admin@example.com",
        "resource_type": "session",
        "status": "failure",
        "ip_address": "10.0.0.5",
        "metadata": {
          "reason": "invalid_password",
          "attempt_number": 3
        },
        "created_at": "2025-01-12T08:15:00Z"
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

## Common Audit Actions

| Action | Description |
|--------|-------------|
| `user.login` | User login attempt |
| `user.logout` | User logout |
| `user.created` | User account created |
| `user.updated` | User profile updated |
| `user.deleted` | User account deleted |
| `user.password_changed` | Password changed |
| `user.password_reset` | Password reset requested |
| `user.activated` | User activated |
| `user.deactivated` | User deactivated |
| `session.refreshed` | Token refreshed |
| `session.revoked` | Session revoked |
| `mfa.challenge` | MFA challenge initiated |
| `mfa.verified` | MFA code verified |
| `mfa.failed` | MFA verification failed |
| `api_key.created` | API key created |
| `api_key.rotated` | API key rotated |
| `api_key.revoked` | API key revoked |
| `org.created` | Organization created |
| `org.updated` | Organization updated |
| `org.deleted` | Organization deleted |
| `org.member_added` | Member added to org |
| `org.member_removed` | Member removed from org |
| `org.ownership_transferred` | Ownership transferred |
| `role.created` | Role created |
| `role.updated` | Role updated |
| `role.deleted` | Role deleted |
| `permission.granted` | Permission granted |
| `permission.revoked` | Permission revoked |
| `invitation.sent` | Invitation sent |
| `invitation.accepted` | Invitation accepted |
| `invitation.declined` | Invitation declined |
| `oauth.login` | OAuth/social login |
| `sso.login` | SSO login |

---

## Data Models

### AuditLog
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `action` | string | Action performed |
| `actor_id` | uuid | User who performed action |
| `actor_email` | string | Actor's email address |
| `resource_type` | string | Type of resource affected |
| `resource_id` | uuid | ID of resource affected |
| `ip_address` | string | Client IP address |
| `user_agent` | string | Client user agent |
| `status` | enum | success, failure |
| `metadata` | object | Additional action context |
| `changes` | object | Before/after values for updates |
| `created_at` | timestamp | When action occurred |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "403",
    "code": "forbidden",
    "title": "Access Denied",
    "detail": "You do not have permission to view audit logs."
  }]
}
```
