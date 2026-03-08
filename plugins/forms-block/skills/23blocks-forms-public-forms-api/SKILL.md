---
name: 23blocks-forms-public-forms-api
description: Handle 23blocks public form access via magic links with optional OTP email verification. Use when accessing forms via magic links, verifying OTP codes, or building public form experiences.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Public Forms API (Magic Links + OTP)

API reference for passwordless form access via magic links with optional OTP verification.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |

## Authentication

No authentication required — uses magic link token:
```
?token=abc123xyz
```

Base URL pattern: `$BLOCKS_API_URL/:url_id/forms/public`

Where `:url_id` is the company's URL identifier (e.g., `starttalking`).

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
