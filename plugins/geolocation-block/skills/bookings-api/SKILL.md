---
name: geolocation-bookings-api
description: Manage 23blocks premise bookings via REST API. Use when creating, updating, cancelling, or deleting bookings for premises within locations, or retrieving user bookings.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Bookings API

Complete API reference for 23blocks premise booking management with scheduling, confirmation, and cancellation workflows.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://geolocation.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Geolocation API base URL | `https://geolocation.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456/bookings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /locations/:unique_id/premises/:premise_unique_id/bookings - List Bookings

Lists all bookings for a specific premise.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456/bookings?page=1&records=20" \
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
      "id": "booking-uuid-789",
      "type": "booking",
      "attributes": {
        "unique_id": "booking-uuid-789",
        "booking_code": "BK-2025-001",
        "premise_id": "premise-uuid-456",
        "user_unique_id": "user-uuid-123",
        "start_time": "2025-02-15T09:00:00Z",
        "end_time": "2025-02-15T10:00:00Z",
        "purpose": "Team standup meeting",
        "attendees": 8,
        "status": "confirmed",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 42
  }
}
```

---

### GET /users/:unique_id/bookings - User Bookings

Retrieves all bookings for a specific user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/bookings?page=1&records=20" \
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
      "id": "booking-uuid-789",
      "type": "booking",
      "attributes": {
        "unique_id": "booking-uuid-789",
        "booking_code": "BK-2025-001",
        "premise_id": "premise-uuid-456",
        "user_unique_id": "user-uuid-123",
        "start_time": "2025-02-15T09:00:00Z",
        "end_time": "2025-02-15T10:00:00Z",
        "purpose": "Team standup meeting",
        "attendees": 8,
        "status": "confirmed"
      },
      "relationships": {
        "premise": {
          "data": { "id": "premise-uuid-456", "type": "premise" }
        },
        "location": {
          "data": { "id": "loc-uuid-123", "type": "location" }
        }
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 18
  }
}
```

---

### POST /locations/:unique_id/premises/:premise_unique_id/bookings - Create Booking

Creates a new booking for a premise.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456/bookings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "booking": {
      "user_unique_id": "user-uuid-123",
      "start_time": "2025-02-15T09:00:00Z",
      "end_time": "2025-02-15T10:00:00Z",
      "purpose": "Team standup meeting",
      "attendees": 8
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User making the booking |
| `start_time` | datetime | Yes | Booking start time (ISO 8601) |
| `end_time` | datetime | Yes | Booking end time (ISO 8601) |
| `purpose` | string | No | Purpose of the booking |
| `attendees` | integer | No | Number of attendees |

**Response 201:**
```json
{
  "data": {
    "id": "booking-uuid-789",
    "type": "booking",
    "attributes": {
      "unique_id": "booking-uuid-789",
      "booking_code": "BK-2025-001",
      "premise_id": "premise-uuid-456",
      "user_unique_id": "user-uuid-123",
      "start_time": "2025-02-15T09:00:00Z",
      "end_time": "2025-02-15T10:00:00Z",
      "purpose": "Team standup meeting",
      "attendees": 8,
      "status": "pending",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Time slot already booked
- `422 Unprocessable Entity` - Validation errors (capacity exceeded, invalid times)

---

### PUT /locations/:unique_id/premises/:premise_unique_id/bookings/:booking_code - Update Booking

Updates an existing booking.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456/bookings/BK-2025-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "booking": {
      "end_time": "2025-02-15T11:00:00Z",
      "attendees": 12,
      "purpose": "Extended team standup meeting"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "booking-uuid-789",
    "type": "booking",
    "attributes": {
      "unique_id": "booking-uuid-789",
      "booking_code": "BK-2025-001",
      "start_time": "2025-02-15T09:00:00Z",
      "end_time": "2025-02-15T11:00:00Z",
      "purpose": "Extended team standup meeting",
      "attendees": 12,
      "status": "confirmed",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Booking not found
- `409 Conflict` - Updated time slot conflicts with existing booking

---

### PUT /locations/:unique_id/premises/:premise_unique_id/bookings/:booking_code/cancel - Cancel Booking

Cancels an existing booking. Preferred over deletion for audit trail.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456/bookings/BK-2025-001/cancel" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "reason": "Meeting rescheduled to next week"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reason` | string | No | Cancellation reason |

**Response 200:**
```json
{
  "data": {
    "id": "booking-uuid-789",
    "type": "booking",
    "attributes": {
      "unique_id": "booking-uuid-789",
      "booking_code": "BK-2025-001",
      "status": "cancelled",
      "cancellation_reason": "Meeting rescheduled to next week",
      "cancelled_at": "2025-01-12T15:00:00Z",
      "updated_at": "2025-01-12T15:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Booking not found
- `422 Unprocessable Entity` - Booking already cancelled

---

### DELETE /locations/:unique_id/premises/:premise_unique_id/bookings/:booking_code - Delete Booking

Permanently deletes a booking.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456/bookings/BK-2025-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Booking not found

---

## Data Models

### Booking
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `booking_code` | string | Human-readable booking code |
| `premise_id` | uuid | Premise being booked |
| `user_unique_id` | uuid | User who made the booking |
| `start_time` | datetime | Booking start time |
| `end_time` | datetime | Booking end time |
| `purpose` | string | Purpose of the booking |
| `attendees` | integer | Number of attendees |
| `status` | enum | pending, confirmed, cancelled |
| `cancellation_reason` | string | Reason for cancellation |
| `cancelled_at` | timestamp | Cancellation time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "409",
    "code": "conflict",
    "title": "Booking Conflict",
    "detail": "The requested time slot conflicts with an existing booking."
  }]
}
```
