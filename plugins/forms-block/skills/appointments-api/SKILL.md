---
name: appointments-api
description: Manage 23blocks appointment scheduling via REST API. Use for booking systems with confirm/cancel workflow and reporting.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Appointments API

Complete API reference for 23blocks appointment management including booking, confirmation, cancellation, and reporting.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://forms.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/appointments/{form_id}/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /appointments/:form_unique_id/instances - List Appointments

Lists all appointments for a specific form.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/appointments/form-123/instances?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `search` | string | No | Search text |
| `sort` | string | No | Sort field (prefix `-` for desc) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "apt-123",
      "type": "appointment",
      "attributes": {
        "unique_id": "apt-123",
        "user_unique_id": "user-456",
        "first_name": "John",
        "last_name": "Doe",
        "email": "john@example.com",
        "phone": "+1-555-0123",
        "appointment_date": "2025-01-15",
        "appointment_time": "14:30",
        "status": "confirmed",
        "notes": "First visit",
        "metadata": {
          "service": "Consultation",
          "duration": 60
        },
        "confirmed_at": "2025-01-10T12:00:00Z",
        "cancelled_at": null,
        "created_at": "2025-01-10T10:30:00Z"
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

### GET /appointments/:form_unique_id/instances/:unique_id - Get Appointment

Retrieves a specific appointment.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/appointments/form-123/instances/apt-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "apt-123",
    "type": "appointment",
    "attributes": {
      "unique_id": "apt-123",
      "user_unique_id": "user-456",
      "first_name": "John",
      "last_name": "Doe",
      "email": "john@example.com",
      "phone": "+1-555-0123",
      "appointment_date": "2025-01-15",
      "appointment_time": "14:30",
      "status": "confirmed",
      "notes": "First visit",
      "confirmation_token": "conf-token-xyz",
      "cancellation_token": "cancel-token-abc",
      "metadata": {
        "service": "Consultation",
        "provider": "Dr. Smith"
      },
      "confirmed_at": "2025-01-10T12:00:00Z",
      "cancelled_at": null,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### POST /appointments/:form_unique_id/instances - Create Appointment

Creates a new appointment booking.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/appointments/form-123/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "appointment": {
      "user_unique_id": "user-456",
      "first_name": "John",
      "last_name": "Doe",
      "email": "john@example.com",
      "phone": "+1-555-0123",
      "appointment_date": "2025-01-15",
      "appointment_time": "14:30",
      "notes": "First visit - consultation",
      "metadata": {
        "service": "Consultation",
        "duration": 60,
        "provider_id": "prov-789"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | string | No | External user ID |
| `first_name` | string | Yes | First name |
| `last_name` | string | Yes | Last name |
| `email` | string | Yes | Email address |
| `phone` | string | No | Phone number |
| `appointment_date` | date | Yes | Appointment date (YYYY-MM-DD) |
| `appointment_time` | time | Yes | Appointment time (HH:MM) |
| `notes` | string | No | Additional notes |
| `metadata` | object | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "new-apt-id",
    "type": "appointment",
    "attributes": {
      "unique_id": "new-apt-id",
      "status": "pending",
      "confirmation_token": "conf-xyz",
      "cancellation_token": "cancel-abc",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /appointments/:form_unique_id/instances/:unique_id - Update Appointment

Updates an existing appointment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/appointments/form-123/instances/apt-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "appointment": {
      "appointment_date": "2025-01-16",
      "appointment_time": "15:00",
      "notes": "Rescheduled - patient request"
    }
  }'
```

---

### POST /appointments/:form_unique_id/instances/:unique_id/confirm - Confirm Appointment

Confirms a pending appointment.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/appointments/form-123/instances/apt-123/confirm" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "apt-123",
    "type": "appointment",
    "attributes": {
      "status": "confirmed",
      "confirmed_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /appointments/:form_unique_id/instances/:unique_id/cancel - Cancel Appointment

Cancels an appointment.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/appointments/form-123/instances/apt-123/cancel" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "reason": "Patient requested cancellation"
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "apt-123",
    "type": "appointment",
    "attributes": {
      "status": "cancelled",
      "cancelled_at": "2025-01-12T10:30:00Z",
      "cancellation_reason": "Patient requested cancellation"
    }
  }
}
```

---

### DELETE /appointments/:form_unique_id/instances/:unique_id - Delete Appointment

Soft-deletes an appointment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/appointments/form-123/instances/apt-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Reporting Endpoints

### POST /reports/appointments/list - Appointment List Report

Generates a list report of appointments with filtering.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/appointments/list" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "form_unique_id": "form-123",
    "start_date": "2025-01-01",
    "end_date": "2025-01-31",
    "status": "confirmed",
    "provider_id": "prov-789"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `form_unique_id` | string | Yes | Form ID |
| `start_date` | date | No | Filter from date |
| `end_date` | date | No | Filter to date |
| `status` | enum | No | Filter by status |
| `provider_id` | string | No | Filter by provider (via metadata) |

**Response 200:**
```json
{
  "data": [...],
  "meta": {
    "total": 45,
    "confirmed": 30,
    "pending": 10,
    "cancelled": 5
  }
}
```

---

### POST /reports/appointments/summary - Appointment Summary Report

Generates a summary report of appointments.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/appointments/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "form_unique_id": "form-123",
    "start_date": "2025-01-01",
    "end_date": "2025-01-31",
    "group_by": "day"
  }'
```

**Group By Options:**
- `day` - Daily breakdown
- `week` - Weekly breakdown
- `month` - Monthly breakdown
- `provider` - By provider (metadata.provider_id)

**Response 200:**
```json
{
  "data": {
    "total_appointments": 150,
    "confirmed": 120,
    "pending": 20,
    "cancelled": 10,
    "confirmation_rate": 0.8,
    "cancellation_rate": 0.067,
    "breakdown": [
      { "date": "2025-01-01", "count": 8, "confirmed": 6 },
      { "date": "2025-01-02", "count": 12, "confirmed": 10 }
    ]
  }
}
```

---

## Data Models

### Appointment
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | string | External user ID |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `email` | string | Email address |
| `phone` | string | Phone number |
| `appointment_date` | date | Appointment date |
| `appointment_time` | time | Appointment time |
| `status` | enum | pending, confirmed, cancelled |
| `notes` | string | Additional notes |
| `confirmation_token` | string | Token for self-confirm |
| `cancellation_token` | string | Token for self-cancel |
| `metadata` | object | Custom metadata |
| `confirmed_at` | timestamp | Confirmation time |
| `cancelled_at` | timestamp | Cancellation time |
| `cancellation_reason` | string | Reason for cancellation |
| `created_at` | timestamp | Creation time |

### Appointment Status
| Status | Description |
|--------|-------------|
| `pending` | Awaiting confirmation |
| `confirmed` | Confirmed by user/admin |
| `cancelled` | Cancelled by user/admin |

---

## Error Codes

| HTTP Status | Code | Description |
|-------------|------|-------------|
| 404 | Not Found | Appointment not found |
| 422 | Unprocessable Entity | Validation error (missing fields) |
| 400 | Bad Request | Invalid date/time format |
| 409 | Conflict | Time slot already booked |

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-forms
```

### Setup

```typescript
import { create23BlocksClient } from '@23blocks/sdk';

const client = create23BlocksClient({
  authToken: process.env.BLOCKS_AUTH_TOKEN!,
  apiKey: process.env.BLOCKS_API_KEY!,
  apiUrl: process.env.BLOCKS_API_URL!,
});
```

### Available Methods

```typescript
// AppointmentsService — client.forms.appointments
list(formUniqueId: string, params?: ListAppointmentsParams): Promise<PageResult<Appointment>>;
get(formUniqueId: string, uniqueId: string): Promise<Appointment>;
create(formUniqueId: string, data: CreateAppointmentRequest): Promise<Appointment>;
update(formUniqueId: string, uniqueId: string, data: UpdateAppointmentRequest): Promise<Appointment>;
delete(formUniqueId: string, uniqueId: string): Promise<void>;
confirm(formUniqueId: string, uniqueId: string): Promise<Appointment>;
cancel(formUniqueId: string, uniqueId: string): Promise<Appointment>;
reportList(data: AppointmentReportRequest): Promise<Appointment[]>;
reportSummary(data: AppointmentReportRequest): Promise<AppointmentReportSummary>;
```

### TypeScript Types

```typescript
import type {
  Appointment,
  CreateAppointmentRequest,
  UpdateAppointmentRequest,
  ListAppointmentsParams,
  AppointmentReportRequest,
  AppointmentReportSummary,
} from '@23blocks/block-forms';
```

### React Hook

```typescript
import { useFormsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFormsBlock();

  // Example: list appointments for a form
  const result = await client.forms.appointments.list('form-unique-id');
}
```
