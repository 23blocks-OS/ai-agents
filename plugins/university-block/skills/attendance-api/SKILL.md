---
name: university-attendance-api
description: Manage 23blocks university attendance and availability via REST API. Use for tracking student and teacher attendance, managing availability slots, and bulk updating schedules.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Attendance API

Complete API reference for 23blocks university attendance tracking and availability slot management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://university.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | University API base URL | `https://university.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/attendance" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Student Attendance

### GET /users/:unique_id/attendance - Student Attendance

Lists attendance records for a student with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/attendance?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Records per page (default: 20) |
| `start_date` | date | No | Filter from date (YYYY-MM-DD) |
| `end_date` | date | No | Filter to date (YYYY-MM-DD) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "attendance-uuid-123",
      "type": "attendance",
      "attributes": {
        "unique_id": "attendance-uuid-123",
        "user_unique_id": "student-uuid-123",
        "user_type": "student",
        "date": "2025-01-15",
        "status": "present",
        "notes": null,
        "created_at": "2025-01-15T09:05:00Z"
      }
    },
    {
      "id": "attendance-uuid-124",
      "type": "attendance",
      "attributes": {
        "unique_id": "attendance-uuid-124",
        "user_unique_id": "student-uuid-123",
        "user_type": "student",
        "date": "2025-01-16",
        "status": "late",
        "notes": "Arrived 15 minutes late",
        "created_at": "2025-01-16T09:20:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 8,
    "totalRecords": 150
  }
}
```

---

### POST /users/:unique_id/attendance - Register Student Attendance

Registers attendance for a student.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/student-uuid-123/attendance" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "attendance": {
      "date": "2025-01-20",
      "status": "present",
      "notes": null
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `date` | date | Yes | Attendance date (YYYY-MM-DD) |
| `status` | enum | Yes | present, absent, late, excused |
| `notes` | string | No | Additional notes |

**Response 201:**
```json
{
  "data": {
    "id": "attendance-uuid-new",
    "type": "attendance",
    "attributes": {
      "unique_id": "attendance-uuid-new",
      "user_unique_id": "student-uuid-123",
      "user_type": "student",
      "date": "2025-01-20",
      "status": "present",
      "notes": null,
      "created_at": "2025-01-20T09:00:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (e.g., duplicate attendance for same date)
- `404 Not Found` - Student not found

---

## Teacher Attendance

### GET /teachers/:unique_id/attendance - Teacher Attendance

Lists attendance records for a teacher with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teachers/teacher-uuid-456/attendance?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Records per page (default: 20) |
| `start_date` | date | No | Filter from date (YYYY-MM-DD) |
| `end_date` | date | No | Filter to date (YYYY-MM-DD) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "attendance-uuid-789",
      "type": "attendance",
      "attributes": {
        "unique_id": "attendance-uuid-789",
        "user_unique_id": "teacher-uuid-456",
        "user_type": "teacher",
        "date": "2025-01-15",
        "status": "present",
        "notes": null,
        "created_at": "2025-01-15T08:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 4,
    "totalRecords": 65
  }
}
```

---

### POST /teachers/:unique_id/attendance - Register Teacher Attendance

Registers attendance for a teacher.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/teachers/teacher-uuid-456/attendance" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "attendance": {
      "date": "2025-01-20",
      "status": "present",
      "notes": "Covering for Prof. Smith"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "attendance-uuid-new",
    "type": "attendance",
    "attributes": {
      "unique_id": "attendance-uuid-new",
      "user_unique_id": "teacher-uuid-456",
      "user_type": "teacher",
      "date": "2025-01-20",
      "status": "present",
      "notes": "Covering for Prof. Smith",
      "created_at": "2025-01-20T08:30:00Z"
    }
  }
}
```

---

## Student Availability

### GET /users/:unique_id/availability - Student Availability Slots

Lists availability slots for a student.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/availability" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "avail-uuid-123",
      "type": "availability",
      "attributes": {
        "unique_id": "avail-uuid-123",
        "user_unique_id": "student-uuid-123",
        "day_of_week": "monday",
        "start_time": "09:00",
        "end_time": "12:00",
        "recurrence": "weekly"
      }
    },
    {
      "id": "avail-uuid-456",
      "type": "availability",
      "attributes": {
        "unique_id": "avail-uuid-456",
        "user_unique_id": "student-uuid-123",
        "day_of_week": "wednesday",
        "start_time": "14:00",
        "end_time": "17:00",
        "recurrence": "weekly"
      }
    }
  ]
}
```

---

### POST /users/:unique_id/availability - Add Student Availability

Adds an availability slot for a student.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/student-uuid-123/availability" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "availability": {
      "day_of_week": "friday",
      "start_time": "10:00",
      "end_time": "13:00",
      "recurrence": "weekly"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `day_of_week` | string | Yes | monday, tuesday, wednesday, thursday, friday, saturday, sunday |
| `start_time` | string | Yes | Start time (HH:MM, 24-hour format) |
| `end_time` | string | Yes | End time (HH:MM, 24-hour format) |
| `recurrence` | string | No | Recurrence pattern (weekly, biweekly) |

**Response 201:**
```json
{
  "data": {
    "id": "avail-uuid-new",
    "type": "availability",
    "attributes": {
      "unique_id": "avail-uuid-new",
      "user_unique_id": "student-uuid-123",
      "day_of_week": "friday",
      "start_time": "10:00",
      "end_time": "13:00",
      "recurrence": "weekly"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (e.g., overlapping slots)

---

### PUT /users/:unique_id/availability/:availability_unique_id - Update Availability

Updates an existing availability slot.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/student-uuid-123/availability/avail-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "availability": {
      "start_time": "10:00",
      "end_time": "13:00"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "avail-uuid-123",
    "type": "availability",
    "attributes": {
      "unique_id": "avail-uuid-123",
      "user_unique_id": "student-uuid-123",
      "day_of_week": "monday",
      "start_time": "10:00",
      "end_time": "13:00",
      "recurrence": "weekly"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Availability slot not found
- `422 Unprocessable Entity` - Validation errors

---

### PUT /users/:unique_id/availabilities/slots - Bulk Update Availability

Bulk updates all availability slots for a student.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/student-uuid-123/availabilities/slots" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "availabilities": [
      {
        "day_of_week": "monday",
        "start_time": "09:00",
        "end_time": "12:00",
        "recurrence": "weekly"
      },
      {
        "day_of_week": "wednesday",
        "start_time": "14:00",
        "end_time": "17:00",
        "recurrence": "weekly"
      },
      {
        "day_of_week": "friday",
        "start_time": "10:00",
        "end_time": "13:00",
        "recurrence": "weekly"
      }
    ]
  }'
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "avail-uuid-new-1",
      "type": "availability",
      "attributes": {
        "unique_id": "avail-uuid-new-1",
        "user_unique_id": "student-uuid-123",
        "day_of_week": "monday",
        "start_time": "09:00",
        "end_time": "12:00",
        "recurrence": "weekly"
      }
    },
    {
      "id": "avail-uuid-new-2",
      "type": "availability",
      "attributes": {
        "unique_id": "avail-uuid-new-2",
        "user_unique_id": "student-uuid-123",
        "day_of_week": "wednesday",
        "start_time": "14:00",
        "end_time": "17:00",
        "recurrence": "weekly"
      }
    },
    {
      "id": "avail-uuid-new-3",
      "type": "availability",
      "attributes": {
        "unique_id": "avail-uuid-new-3",
        "user_unique_id": "student-uuid-123",
        "day_of_week": "friday",
        "start_time": "10:00",
        "end_time": "13:00",
        "recurrence": "weekly"
      }
    }
  ]
}
```

---

### DELETE /users/:unique_id/availability/:availability_unique_id - Delete Availability

Deletes a specific availability slot.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/student-uuid-123/availability/avail-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### DELETE /users/:unique_id/availability - Delete All Availability

Deletes all availability slots for a student.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/student-uuid-123/availability" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Attendance
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | User unique ID |
| `user_type` | enum | student, teacher |
| `date` | date | Attendance date |
| `status` | enum | present, absent, late, excused |
| `notes` | string | Additional notes |
| `created_at` | timestamp | Creation time |

### Availability
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | User unique ID |
| `day_of_week` | string | monday, tuesday, wednesday, thursday, friday, saturday, sunday |
| `start_time` | string | Start time (HH:MM) |
| `end_time` | string | End time (HH:MM) |
| `recurrence` | string | weekly, biweekly |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Attendance record already exists for this date."
  }]
}
```
