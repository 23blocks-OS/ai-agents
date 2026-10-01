---
name: 23blocks-forms-public-forms-api
description: "Forms Block public access by magic link with optional email OTP. Use when an unauthenticated person opens a form."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Public Forms API (Magic Links + OTP)

API reference for passwordless form access via magic links with optional OTP verification.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://forms.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/:url_id/forms/public` | Get form; returns pending (OTP required) or verified response with fields |
| POST | `/:url_id/forms/public/send-otp` | Send 6-digit OTP code to assignee email |
| POST | `/:url_id/forms/public/verify-otp` | Verify OTP code; returns full form data on success |
| POST | `/:url_id/forms/public` | Submit completed form responses |
| PATCH | `/:url_id/forms/public` | Save draft responses without submitting |

---

## Data Models

### Response Array Format

Responses are indexed by field position in the schema:

```
Schema fields:
  [0] intro (display) - No response (null)
  [1] q1 (radio) - Index of selected option
  [2] q2 (radio) - Index of selected option
  [3] notes (text) - Text response

Responses array:
  [null, 2, 1, "Patient notes here"]
```

### Email Masking

Email addresses are masked for privacy:
```
john.doe@example.com → j***e@e***e.com
test@company.org → t**t@c*****y.org
```

### OTP Settings
| Setting | Value |
|---------|-------|
| OTP Length | 6 digits |
| Expiration | 10 minutes |
| Max Attempts | 5 |
| Resend Cooldown | 60 seconds |

## Error Response Format

```json
{
  "error": "Human-readable message",
  "code": "ERROR_CODE",
  "retry_after": 45,
  "attempts_remaining": 3
}
```

Or for validation errors:
```json
{ "errors": ["Field 'email' must be a valid email address", "Field 'q1' is required"] }
```

---

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-forms`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// ApplicationFormsService — client.forms.applicationForms
get(urlId: string): Promise<ApplicationForm>;
submit(urlId: string, data: ApplicationFormSubmission): Promise<ApplicationFormResponse>;
draft(urlId: string, data: ApplicationFormDraft): Promise<ApplicationFormResponse>;
sendOtp(urlId: string): Promise<SendOtpResponse>;
verifyOtp(urlId: string, data: VerifyOtpRequest): Promise<ApplicationForm>;
```

### TypeScript Types

```typescript
import type {
  ApplicationForm,
  ApplicationFormSubmission,
  ApplicationFormDraft,
  ApplicationFormResponse,
  SendOtpResponse,
  VerifyOtpRequest,
  VerificationStatus,
  OtpErrorCode,
  OtpError,
} from '@23blocks/block-forms';
```

### React Hook

```typescript
import { useFormsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFormsBlock();

  // Example: get a public form via magic link
  const result = await client.forms.applicationForms.get('my-company-url');
}
```
