---
name: 23blocks-forms-sdk
description: TypeScript SDK for 23blocks Forms API. Use for type definitions, API client setup, and React integration patterns.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Forms SDK (TypeScript/React)

TypeScript types, API client, and React patterns for 23blocks Forms API integration.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

### Basic Configuration

```typescript
import { FormsClient } from '@23blocks/forms-sdk';

// CRITICAL: Verify env vars before initializing
if (!process.env.BLOCKS_API_URL || !process.env.BLOCKS_AUTH_TOKEN || !process.env.BLOCKS_API_KEY) {
  throw new Error('Missing required env vars: BLOCKS_API_URL, BLOCKS_AUTH_TOKEN, BLOCKS_API_KEY');
}

const client = new FormsClient({
  baseUrl: process.env.BLOCKS_API_URL,
  accessToken: process.env.BLOCKS_AUTH_TOKEN,
  appId: process.env.BLOCKS_API_KEY,
});
```

### With Auth Provider Integration

```typescript
import { FormsClient } from '@23blocks/forms-sdk';
import { useAuth } from '@23blocks/auth-sdk';

function useFormsClient() {
  const { accessToken, appId } = useAuth();

  if (!process.env.BLOCKS_API_URL) {
    throw new Error('Missing required env var: BLOCKS_API_URL');
  }

  return new FormsClient({
    baseUrl: process.env.BLOCKS_API_URL,
    accessToken,
    appId,
  });
}
```

## SDK Methods

> Full SDK method documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Category | Method | Description |
|----------|--------|-------------|
| Forms CRUD | `client.forms.list(params)` | List forms with pagination and type filter |
| Forms CRUD | `client.forms.get(formId)` | Get single form |
| Forms CRUD | `client.forms.create(data)` | Create form |
| Forms CRUD | `client.forms.update(formId, data)` | Update form |
| Forms CRUD | `client.forms.delete(formId)` | Delete form |
| Landing | `client.landings.create(formId, data)` | Create landing submission |
| Landing | `client.landings.list(formId, params)` | List submissions |
| Appointments | `client.appointments.create(formId, data)` | Create appointment |
| Appointments | `client.appointments.confirm(formId, aptId)` | Confirm appointment |
| Appointments | `client.appointments.cancel(formId, aptId, data)` | Cancel appointment |

| Hook | Parameters | Description |
|------|------------|-------------|
| `useForm` | `{ client, formId }` | Load single form with loading/error state |
| `useForms` | `{ client, page?, perPage?, formType? }` | List forms with pagination |
| `useCreateForm` | `{ client, onSuccess?, onError? }` | Create form mutation with callbacks |

---

## Data Models

### Form
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Form name |
| `form_type` | enum | landing, survey, appointment, subscription, referral, app_form |
| `status` | enum | active, inactive, archived |
| `settings` | object | allow_multiple_submissions, require_authentication, notification_email, redirect_url, confirmation_message |
| `form_schema_id` | uuid | Schema ID (for app_form type) |

### Form Instance Types by Form Type
| Form Type | Instance Type | Key Fields |
|-----------|---------------|------------|
| `landing` | `LandingInstance` | first_name, last_name, email, company, message, utm_*, payload |
| `survey` | `SurveyInstance` | assigned_to_email, status, responses, magic_link_token |
| `appointment` | `Appointment` | first_name, last_name, email, appointment_date, appointment_time, status |
| `subscription` | `Subscription` | email, status (active/unsubscribed), preferences |
| `referral` | `Referral` | referrer_email, referred_email, status, reward_status |

### Pagination Meta
```typescript
{ totalPages: number; totalRecords: number; currentPage?: number; perPage?: number; }
```

## Error Response Format

```json
{ "errors": [{ "status": "422", "code": "validation_error", "title": "Validation Failed", "detail": "..." }] }
```

## SDK Usage (TypeScript)

```bash
npm install @23blocks/forms-sdk
# or
yarn add @23blocks/forms-sdk
```

See [ENDPOINTS.md](ENDPOINTS.md) for full type definitions, React hooks, client method examples, React Query integration, error handling patterns, and constants/utilities.
