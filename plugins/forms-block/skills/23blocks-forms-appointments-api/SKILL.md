---
name: 23blocks-forms-appointments-api
description: "Forms Block appointments: book, confirm, cancel, list and summary reports. Use for booking flows."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Appointments API

Complete API reference for 23blocks appointment management including booking, confirmation, cancellation, and reporting.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://forms.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

Request and response detail for each endpoint: [ENDPOINTS.md](ENDPOINTS.md).

| Method | Path | Description |
|--------|------|-------------|
| GET | `/appointments/:form_unique_id/instances` | List Appointments |
| GET | `/appointments/:form_unique_id/instances/:unique_id` | Get Appointment |
| POST | `/appointments/:form_unique_id/instances` | Create Appointment |
| PUT | `/appointments/:form_unique_id/instances/:unique_id` | Update Appointment |
| POST | `/appointments/:form_unique_id/instances/:unique_id/confirm` | Confirm Appointment |
| POST | `/appointments/:form_unique_id/instances/:unique_id/cancel` | Cancel Appointment |
| DELETE | `/appointments/:form_unique_id/instances/:unique_id` | Delete Appointment |
| POST | `/reports/appointments/list` | Appointment List Report (Reporting Endpoints) |
| POST | `/reports/appointments/summary` | Appointment Summary Report (Reporting Endpoints) |

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

## Error Codes

| HTTP Status | Code | Description |
|-------------|------|-------------|
| 404 | Not Found | Appointment not found |
| 422 | Unprocessable Entity | Validation error (missing fields) |
| 400 | Bad Request | Invalid date/time format |
| 409 | Conflict | Time slot already booked |

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-forms`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
