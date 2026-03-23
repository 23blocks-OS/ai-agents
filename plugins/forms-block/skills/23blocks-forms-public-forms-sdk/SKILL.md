---
name: 23blocks-forms-public-forms-sdk
description: TypeScript SDK for 23blocks public forms with magic link access and OTP verification. Use for building secure, unauthenticated form experiences.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Public Forms SDK (TypeScript/React)

TypeScript types, React components, and patterns for magic link access and OTP verification flows.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://forms.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://forms.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Overview

Public forms allow unauthenticated access via magic links with optional OTP verification:
- **Magic Links**: Token-based access without authentication
- **OTP Verification**: Optional email OTP for sensitive forms (HIPAA, etc.)
- **Draft Saving**: Save progress before final submission
- **Status Tracking**: Pending → Verified → In Progress → Completed

## SDK Methods

> Full SDK method documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Class / Hook | Method | Description |
|---|---|---|
| `PublicFormsClient` | `getForm(urlId, token)` | Get form; returns pending (OTP required) or verified response |
| `PublicFormsClient` | `sendOtp(urlId, token)` | Send 6-digit OTP code to assignee email |
| `PublicFormsClient` | `verifyOtp(urlId, token, code)` | Verify OTP; returns full form data on success |
| `PublicFormsClient` | `saveDraft(urlId, token, responses)` | Save draft responses without submitting |
| `PublicFormsClient` | `submit(urlId, token, responses)` | Submit completed form responses |
| `usePublicForm` | `{ urlId, token }` | Hook managing OTP flow + form state (sendOtp, verifyOtp, saveDraft, submit) |

| Component | Props | Description |
|-----------|-------|-------------|
| `PublicFormPage` | `urlId, token` | Top-level page: routes to OTP screen or form based on state |
| `OtpVerificationScreen` | `formName, maskedEmail, otpSent, sendOtp, verifyOtp, ...` | Two-step OTP send/verify UI |
| `OtpInput` | `value, onChange, length?, disabled?` | 6-digit OTP input with auto-focus and paste support |
| `CompletedScreen` | `completedAt` | Success screen shown after submission |
| `ExpiredScreen` | — | Screen shown when magic link has expired |
| `ErrorCard` | `error` | Error screen with human-readable messages by status code |

---

## Data Models

### PublicFormVerifiedResponse
Returned when OTP not required or already verified. Includes `data` (FormInstance with `verification_status: 'verified'`) and `included` (FormSchemaVersion with form fields).

### PublicFormPendingResponse
Returned when OTP is required but not yet verified. Includes `verification_status: 'pending'`, `masked_email`, `otp_sent`, `form_name`. No form fields exposed.

### OTP Settings
| Setting | Value |
|---------|-------|
| Code length | 6 digits |
| Expiration | 10 minutes (600 seconds) |
| Max attempts | 5 |
| Resend cooldown | 60 seconds |

### Magic Link URL Format
```
/{urlId}/forms/public?token={token}
```

## Error Response Format

```json
{ "error": "Human-readable message", "code": "ERROR_CODE", "attempts_remaining": 3 }
```

Common codes: `INVALID_OTP`, `OTP_EXPIRED`, `MAX_ATTEMPTS_EXCEEDED`, `TOKEN_REVOKED`, `TOKEN_EXPIRED`

## SDK Usage (TypeScript)

```bash
npm install @23blocks/block-forms
```

```typescript
import { create23BlocksClient } from '@23blocks/sdk';

const client = create23BlocksClient({
  authToken: process.env.BLOCKS_AUTH_TOKEN!,
  apiKey: process.env.BLOCKS_API_KEY!,
  apiUrl: process.env.BLOCKS_API_URL!,
});

// client.forms.applicationForms.get(urlId)
// client.forms.applicationForms.sendOtp(urlId)
// client.forms.applicationForms.verifyOtp(urlId, { code })
// client.forms.applicationForms.submit(urlId, data)
// client.forms.applicationForms.draft(urlId, data)
```

See [ENDPOINTS.md](ENDPOINTS.md) for full type definitions, React components, hooks, Next.js integration, utility functions, and testing utilities.
