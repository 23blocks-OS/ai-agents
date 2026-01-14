---
name: public-forms-api
description: Handle 23blocks public form access via magic links with optional OTP email verification. No authentication required - uses token-based access.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Public Forms API (Magic Links + OTP)

API reference for passwordless form access via magic links with optional OTP verification.

## Required Environment Variables

**BEFORE making ANY API call**, verify the API URL environment variable is set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ]; then
  echo "ERROR: Missing BLOCKS_API_URL environment variable"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://forms.api.us.23blocks.com)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |

## Base URL
```bash
$BLOCKS_API_URL/:url_id/forms/public
```

Where `:url_id` is the company's URL identifier (e.g., `starttalking`).

## Authentication
No authentication required - uses magic link token:
```
?token=abc123xyz
```

---

## OTP Verification Flow

For forms with `require_otp_verification: true`:

```
1. GET /forms/public?token=xxx
   → Returns verification_status: "pending"
   → Shows masked email, form name, no fields

2. POST /forms/public/send-otp?token=xxx
   → Sends 6-digit OTP code to email
   → Returns expires_in: 600 (10 minutes)

3. POST /forms/public/verify-otp?token=xxx
   → Verifies OTP code
   → Returns full form data if correct

4. POST /forms/public?token=xxx
   → Submit completed form
   → Token revoked after submission
```

---

## Endpoints

### GET /:url_id/forms/public - Get Form

Retrieves form for magic link access. Returns limited data if OTP verification is pending.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mycompany/forms/public?token=abc123xyz"
```

**Response 200 (OTP Required, Not Verified):**
```json
{
  "data": {
    "type": "FormInstance",
    "id": "inst-123",
    "attributes": {
      "verification_status": "pending",
      "assigned_to_name": "John Doe",
      "masked_email": "j***e@e***e.com",
      "otp_sent": false,
      "otp_expires_in": null,
      "form_name": "Patient Intake Form",
      "can_resend": true,
      "resend_available_in": 0
    }
  }
}
```

**Response 200 (Verified or OTP Not Required):**
```json
{
  "data": {
    "type": "FormInstance",
    "id": "inst-123",
    "attributes": {
      "verification_status": "verified",
      "unique_id": "inst-123",
      "status": "pending",
      "assigned_to_name": "John Doe",
      "assigned_to_email": "john@example.com",
      "responses": null,
      "started_at": null
    }
  },
  "included": [
    {
      "type": "FormSchemaVersion",
      "id": "version-id",
      "attributes": {
        "form_fields": [
          {
            "name": "intro",
            "type": "display",
            "content": "Please complete this form..."
          },
          {
            "name": "q1",
            "type": "radio",
            "label": "How are you feeling today?",
            "required": true,
            "options": ["Great", "Good", "Okay", "Not well"]
          }
        ],
        "validation_rules": {...},
        "scoring_rules": {...}
      }
    }
  ]
}
```

**Errors:**
- `401 Unauthorized` - Token missing
- `401 Unauthorized` - Invalid token
- `403 TOKEN_REVOKED` - Form already submitted
- `403 TOKEN_EXPIRED` - Magic link expired
- `403 ALREADY_COMPLETED` - Form already completed

---

### POST /:url_id/forms/public/send-otp - Send OTP Code

Generates and sends a 6-digit OTP code to the assignee's email.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mycompany/forms/public/send-otp?token=abc123xyz"
```

**Response 200:**
```json
{
  "message": "Verification code sent",
  "masked_email": "j***e@e***e.com",
  "expires_in": 600,
  "sent_count": 1
}
```

**Errors:**
- `400 OTP_NOT_REQUIRED` - Form doesn't require OTP
- `400 ALREADY_VERIFIED` - Email already verified
- `429 RATE_LIMITED` - Too many requests (retry after 60 seconds)

```json
{
  "error": "Please wait before requesting another code",
  "code": "RATE_LIMITED",
  "retry_after": 45
}
```

---

### POST /:url_id/forms/public/verify-otp - Verify OTP Code

Verifies the OTP code and returns full form data if correct.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mycompany/forms/public/verify-otp?token=abc123xyz" \
  -H "Content-Type: application/json" \
  -d '{"code": "123456"}'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | 6-digit OTP code |

**Response 200 (Success):**
```json
{
  "data": {
    "type": "FormInstance",
    "id": "inst-123",
    "attributes": {
      "verification_status": "verified",
      "unique_id": "inst-123",
      "status": "pending",
      "assigned_to_email": "john@example.com"
    }
  },
  "included": [
    {
      "type": "FormSchemaVersion",
      "attributes": {
        "form_fields": [...]
      }
    }
  ]
}
```

**Response 422 (Invalid Code):**
```json
{
  "error": "Invalid verification code",
  "code": "INVALID_CODE",
  "attempts_remaining": 4
}
```

**Errors:**
| Code | HTTP Status | Description |
|------|-------------|-------------|
| `OTP_NOT_REQUIRED` | 400 | Form doesn't require OTP |
| `CODE_REQUIRED` | 400 | Code parameter missing |
| `ALREADY_VERIFIED` | 200 | Already verified (returns form data) |
| `CODE_EXPIRED` | 422 | OTP has expired (request new code) |
| `INVALID_CODE` | 422 | Wrong code (attempts remaining shown) |
| `ATTEMPTS_EXCEEDED` | 403 | Max 5 attempts exceeded |

---

### POST /:url_id/forms/public - Submit Form

Submits completed form responses. Requires OTP verification if enabled.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mycompany/forms/public?token=abc123xyz" \
  -H "Content-Type: application/json" \
  -d '{
    "responses": [null, 2, 1, 3, "Additional notes here"]
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `responses` | array | Yes | Array of responses by field index |

**Response 200:**
```json
{
  "message": "Form submitted successfully",
  "instance_id": "inst-123"
}
```

**Errors:**
- `403 OTP_REQUIRED` - Email verification required before submitting
- `403 TOKEN_REVOKED` - Token already used
- `403 TOKEN_EXPIRED` - Token expired
- `422 Unprocessable Entity` - Validation errors

```json
{
  "errors": ["Field 'q1' is required", "Field 'q3' must be at least 10 characters"]
}
```

---

### PATCH /:url_id/forms/public - Save Draft

Auto-saves form responses without submitting.

**Request:**
```bash
curl -X PATCH "$BLOCKS_API_URL/mycompany/forms/public?token=abc123xyz" \
  -H "Content-Type: application/json" \
  -d '{
    "responses": [null, 2, 1, null, null]
  }'
```

**Response 200:**
```json
{
  "message": "Draft saved successfully",
  "instance_id": "inst-123"
}
```

**Errors:**
- `403 OTP_REQUIRED` - Email verification required before saving
- `403 TOKEN_REVOKED` - Token revoked
- `403 TOKEN_EXPIRED` - Token expired

---

## OTP Configuration

### Enable OTP for a Form
```bash
curl -X PUT "$BLOCKS_API_URL/forms/form-id" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "form": {
      "require_otp_verification": true
    }
  }'
```

### OTP Settings
| Setting | Value | Description |
|---------|-------|-------------|
| OTP Length | 6 digits | Standard 6-digit code |
| Expiration | 10 minutes | Code expires after 10 minutes |
| Max Attempts | 5 | Max verification attempts |
| Resend Cooldown | 60 seconds | Wait time between sends |

---

## Response Array Format

Responses are indexed by field position in the schema:

```
Schema fields:
  [0] intro (display) - No response (null)
  [1] q1 (radio) - Index of selected option
  [2] q2 (radio) - Index of selected option
  [3] notes (text) - Text response

Responses array:
  [null, 2, 1, "Patient notes here"]
   ^     ^  ^  ^
   |     |  |  +-- Text field value
   |     |  +-- Option index 1 (e.g., "Several days")
   |     +-- Option index 2 (e.g., "More than half")
   +-- Display field (always null)
```

---

## Email Masking

Email addresses are masked for privacy:
```
john.doe@example.com → j***e@e***e.com
test@company.org → t**t@c*****y.org
```

---

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
{
  "errors": [
    "Field 'email' must be a valid email address",
    "Field 'q1' is required"
  ]
}
```

---

## Security Considerations

1. **Token Security**
   - Tokens are single-use (revoked after submission)
   - Tokens have configurable expiration
   - Tokens are cryptographically random

2. **OTP Security**
   - 6-digit TOTP codes (RFC 6238)
   - 10-minute expiration
   - Max 5 verification attempts
   - 60-second rate limiting between sends

3. **Data Protection**
   - Email addresses masked in responses
   - No authentication headers in public endpoints
   - Form fields hidden until OTP verified

---

## Frontend Integration Example

```typescript
// Check form status
const response = await fetch(`${baseUrl}/${urlId}/forms/public?token=${token}`);
const data = await response.json();

if (data.data.attributes.verification_status === 'pending') {
  // Show OTP verification UI
  showOtpScreen(data.data.attributes);
} else {
  // Show form fields
  renderForm(data.included[0].attributes.form_fields);
}

// Send OTP
async function sendOtp() {
  const response = await fetch(
    `${baseUrl}/${urlId}/forms/public/send-otp?token=${token}`,
    { method: 'POST' }
  );
  return response.json();
}

// Verify OTP
async function verifyOtp(code: string) {
  const response = await fetch(
    `${baseUrl}/${urlId}/forms/public/verify-otp?token=${token}`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ code })
    }
  );
  return response.json();
}

// Submit form
async function submitForm(responses: any[]) {
  const response = await fetch(
    `${baseUrl}/${urlId}/forms/public?token=${token}`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ responses })
    }
  );
  return response.json();
}
```
