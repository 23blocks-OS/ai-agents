---
name: public-forms-sdk
description: TypeScript SDK for 23blocks public forms with magic link access and OTP verification. Use for building secure, unauthenticated form experiences.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Public Forms SDK (TypeScript/React)

TypeScript types, React components, and patterns for magic link access and OTP verification flows.

## Overview

Public forms allow unauthenticated access via magic links with optional OTP verification:
- **Magic Links**: Token-based access without authentication
- **OTP Verification**: Optional email OTP for sensitive forms (HIPAA, etc.)
- **Draft Saving**: Save progress before final submission
- **Status Tracking**: Pending → Verified → In Progress → Completed

---

## Type Definitions

### Public Form Response Types

```typescript
// Response when OTP is NOT required (or already verified)
interface PublicFormVerifiedResponse {
  data: {
    id: string;
    type: 'FormInstance';
    attributes: {
      unique_id: string;
      verification_status: 'verified';
      assigned_to_name: string | null;
      assigned_to_email: string;
      status: FormInstanceStatus;
      responses: unknown[];
      expires_at: string | null;
      started_at: string | null;
      completed_at: string | null;
      created_at: string;
    };
  };
  included: FormSchemaVersion[];
}

// Response when OTP IS required but NOT yet verified
interface PublicFormPendingResponse {
  data: {
    id: string;
    type: 'FormInstance';
    attributes: {
      verification_status: 'pending';
      assigned_to_name: string | null;
      masked_email: string;        // e.g., "p***o@e***.com"
      otp_sent: boolean;
      form_name: string;
    };
  };
}

type PublicFormResponse = PublicFormVerifiedResponse | PublicFormPendingResponse;

type FormInstanceStatus =
  | 'pending'
  | 'in_progress'
  | 'completed'
  | 'cancelled'
  | 'expired';

// OTP send response
interface OtpSendResponse {
  message: string;
  masked_email: string;
  expires_in: number;  // seconds (600 = 10 minutes)
}

// OTP verify success (returns full form)
type OtpVerifySuccessResponse = PublicFormVerifiedResponse;

// OTP verify failure
interface OtpVerifyFailureResponse {
  error: string;
  code: 'INVALID_OTP' | 'OTP_EXPIRED' | 'MAX_ATTEMPTS_EXCEEDED';
  attempts_remaining?: number;
}
```

### Magic Link Types

```typescript
interface MagicLinkParams {
  urlId: string;      // Company URL identifier
  token: string;      // Magic link token
}

// Extract from URL: /company-url/forms/public?token=xxx
function parseMagicLink(url: string): MagicLinkParams {
  const urlObj = new URL(url);
  const pathParts = urlObj.pathname.split('/');
  const urlId = pathParts[1];  // First segment after /
  const token = urlObj.searchParams.get('token');

  if (!urlId || !token) {
    throw new Error('Invalid magic link URL');
  }

  return { urlId, token };
}
```

---

## Public Forms Client

```typescript
class PublicFormsClient {
  private baseUrl: string;

  constructor(baseUrl = 'https://forms.23blocks.com') {
    this.baseUrl = baseUrl;
  }

  // Get public form (checks OTP requirement)
  async getForm(urlId: string, token: string): Promise<PublicFormResponse> {
    const response = await fetch(
      `${this.baseUrl}/${urlId}/forms/public?token=${token}`
    );

    if (!response.ok) {
      throw await this.handleError(response);
    }

    return response.json();
  }

  // Send OTP verification code
  async sendOtp(urlId: string, token: string): Promise<OtpSendResponse> {
    const response = await fetch(
      `${this.baseUrl}/${urlId}/forms/public/send-otp?token=${token}`,
      { method: 'POST' }
    );

    if (!response.ok) {
      throw await this.handleError(response);
    }

    return response.json();
  }

  // Verify OTP code
  async verifyOtp(
    urlId: string,
    token: string,
    code: string
  ): Promise<OtpVerifySuccessResponse> {
    const response = await fetch(
      `${this.baseUrl}/${urlId}/forms/public/verify-otp?token=${token}`,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ code }),
      }
    );

    if (!response.ok) {
      const error = await response.json() as OtpVerifyFailureResponse;
      throw error;
    }

    return response.json();
  }

  // Save draft responses
  async saveDraft(
    urlId: string,
    token: string,
    responses: unknown[]
  ): Promise<PublicFormVerifiedResponse> {
    const response = await fetch(
      `${this.baseUrl}/${urlId}/forms/public/draft?token=${token}`,
      {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ responses }),
      }
    );

    if (!response.ok) {
      throw await this.handleError(response);
    }

    return response.json();
  }

  // Submit form
  async submit(
    urlId: string,
    token: string,
    responses: unknown[]
  ): Promise<PublicFormVerifiedResponse> {
    const response = await fetch(
      `${this.baseUrl}/${urlId}/forms/public/submit?token=${token}`,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ responses }),
      }
    );

    if (!response.ok) {
      throw await this.handleError(response);
    }

    return response.json();
  }

  private async handleError(response: Response): Promise<never> {
    const error = await response.json();
    throw {
      status: response.status,
      ...error,
    };
  }
}
```

---

## React Hooks

### usePublicForm Hook

```typescript
import { useState, useEffect, useCallback } from 'react';

interface UsePublicFormOptions {
  urlId: string;
  token: string;
}

interface UsePublicFormResult {
  // State
  loading: boolean;
  error: unknown | null;
  requiresOtp: boolean;
  otpSent: boolean;
  maskedEmail: string | null;
  formName: string | null;
  isVerified: boolean;
  formData: PublicFormVerifiedResponse | null;

  // Actions
  sendOtp: () => Promise<void>;
  verifyOtp: (code: string) => Promise<boolean>;
  saveDraft: (responses: unknown[]) => Promise<void>;
  submit: (responses: unknown[]) => Promise<void>;

  // OTP state
  otpError: string | null;
  attemptsRemaining: number | null;
  sendingOtp: boolean;
  verifyingOtp: boolean;
}

function usePublicForm({ urlId, token }: UsePublicFormOptions): UsePublicFormResult {
  const [client] = useState(() => new PublicFormsClient());

  // Loading states
  const [loading, setLoading] = useState(true);
  const [sendingOtp, setSendingOtp] = useState(false);
  const [verifyingOtp, setVerifyingOtp] = useState(false);

  // Data states
  const [error, setError] = useState<unknown | null>(null);
  const [requiresOtp, setRequiresOtp] = useState(false);
  const [otpSent, setOtpSent] = useState(false);
  const [maskedEmail, setMaskedEmail] = useState<string | null>(null);
  const [formName, setFormName] = useState<string | null>(null);
  const [formData, setFormData] = useState<PublicFormVerifiedResponse | null>(null);

  // OTP error handling
  const [otpError, setOtpError] = useState<string | null>(null);
  const [attemptsRemaining, setAttemptsRemaining] = useState<number | null>(null);

  const isVerified = formData !== null;

  // Fetch form on mount
  useEffect(() => {
    async function fetchForm() {
      setLoading(true);
      setError(null);

      try {
        const response = await client.getForm(urlId, token);

        if (response.data.attributes.verification_status === 'pending') {
          // OTP required
          const pending = response as PublicFormPendingResponse;
          setRequiresOtp(true);
          setOtpSent(pending.data.attributes.otp_sent);
          setMaskedEmail(pending.data.attributes.masked_email);
          setFormName(pending.data.attributes.form_name);
        } else {
          // Already verified or OTP not required
          setFormData(response as PublicFormVerifiedResponse);
        }
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    }

    fetchForm();
  }, [urlId, token]);

  // Send OTP
  const sendOtp = useCallback(async () => {
    setSendingOtp(true);
    setOtpError(null);

    try {
      const response = await client.sendOtp(urlId, token);
      setOtpSent(true);
      setMaskedEmail(response.masked_email);
    } catch (err) {
      setOtpError('Failed to send verification code. Please try again.');
    } finally {
      setSendingOtp(false);
    }
  }, [urlId, token]);

  // Verify OTP
  const verifyOtp = useCallback(async (code: string): Promise<boolean> => {
    setVerifyingOtp(true);
    setOtpError(null);

    try {
      const response = await client.verifyOtp(urlId, token, code);
      setFormData(response);
      setRequiresOtp(false);
      return true;
    } catch (err) {
      const error = err as OtpVerifyFailureResponse;

      if (error.code === 'MAX_ATTEMPTS_EXCEEDED') {
        setOtpError('Maximum attempts exceeded. Please request a new code.');
        setAttemptsRemaining(0);
      } else if (error.code === 'OTP_EXPIRED') {
        setOtpError('Code has expired. Please request a new code.');
      } else {
        setOtpError('Invalid code. Please try again.');
        setAttemptsRemaining(error.attempts_remaining ?? null);
      }
      return false;
    } finally {
      setVerifyingOtp(false);
    }
  }, [urlId, token]);

  // Save draft
  const saveDraft = useCallback(async (responses: unknown[]) => {
    const response = await client.saveDraft(urlId, token, responses);
    setFormData(response);
  }, [urlId, token]);

  // Submit form
  const submit = useCallback(async (responses: unknown[]) => {
    const response = await client.submit(urlId, token, responses);
    setFormData(response);
  }, [urlId, token]);

  return {
    loading,
    error,
    requiresOtp,
    otpSent,
    maskedEmail,
    formName,
    isVerified,
    formData,
    sendOtp,
    verifyOtp,
    saveDraft,
    submit,
    otpError,
    attemptsRemaining,
    sendingOtp,
    verifyingOtp,
  };
}
```

---

## React Components

### PublicFormPage Component

```typescript
import React from 'react';

interface PublicFormPageProps {
  urlId: string;
  token: string;
}

function PublicFormPage({ urlId, token }: PublicFormPageProps) {
  const {
    loading,
    error,
    requiresOtp,
    isVerified,
    formData,
    // OTP methods
    sendOtp,
    verifyOtp,
    otpSent,
    maskedEmail,
    formName,
    otpError,
    attemptsRemaining,
    sendingOtp,
    verifyingOtp,
    // Form methods
    saveDraft,
    submit,
  } = usePublicForm({ urlId, token });

  // Loading state
  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <LoadingSpinner />
      </div>
    );
  }

  // Error state
  if (error) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <ErrorCard error={error} />
      </div>
    );
  }

  // OTP verification required
  if (requiresOtp && !isVerified) {
    return (
      <OtpVerificationScreen
        formName={formName}
        maskedEmail={maskedEmail}
        otpSent={otpSent}
        sendOtp={sendOtp}
        verifyOtp={verifyOtp}
        error={otpError}
        attemptsRemaining={attemptsRemaining}
        sendingOtp={sendingOtp}
        verifyingOtp={verifyingOtp}
      />
    );
  }

  // Form verified - show form
  if (formData) {
    const instance = formData.data;
    const schema = formData.included?.[0];

    // Already completed
    if (instance.attributes.status === 'completed') {
      return <CompletedScreen completedAt={instance.attributes.completed_at} />;
    }

    // Expired
    if (instance.attributes.status === 'expired') {
      return <ExpiredScreen />;
    }

    // Show form
    return (
      <PublicFormRenderer
        instance={instance}
        schema={schema}
        onSave={saveDraft}
        onSubmit={submit}
      />
    );
  }

  return null;
}
```

### OTP Verification Screen

```typescript
interface OtpVerificationScreenProps {
  formName: string | null;
  maskedEmail: string | null;
  otpSent: boolean;
  sendOtp: () => Promise<void>;
  verifyOtp: (code: string) => Promise<boolean>;
  error: string | null;
  attemptsRemaining: number | null;
  sendingOtp: boolean;
  verifyingOtp: boolean;
}

function OtpVerificationScreen({
  formName,
  maskedEmail,
  otpSent,
  sendOtp,
  verifyOtp,
  error,
  attemptsRemaining,
  sendingOtp,
  verifyingOtp,
}: OtpVerificationScreenProps) {
  const [code, setCode] = useState('');

  const handleSendOtp = async () => {
    await sendOtp();
  };

  const handleVerify = async (e: React.FormEvent) => {
    e.preventDefault();
    if (code.length === 6) {
      await verifyOtp(code);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50 p-4">
      <div className="bg-white rounded-lg shadow-lg max-w-md w-full p-8">
        {/* Header */}
        <div className="text-center mb-8">
          <div className="w-16 h-16 bg-blue-100 rounded-full flex items-center justify-center mx-auto mb-4">
            <LockIcon className="w-8 h-8 text-blue-600" />
          </div>
          <h1 className="text-2xl font-bold text-gray-900">Verify Your Identity</h1>
          {formName && (
            <p className="text-gray-600 mt-2">
              To access <strong>{formName}</strong>
            </p>
          )}
        </div>

        {!otpSent ? (
          // Step 1: Send OTP
          <div className="text-center">
            <p className="text-gray-600 mb-6">
              We'll send a verification code to your email
              {maskedEmail && (
                <span className="block font-medium mt-1">{maskedEmail}</span>
              )}
            </p>
            <button
              onClick={handleSendOtp}
              disabled={sendingOtp}
              className="w-full btn-primary py-3"
            >
              {sendingOtp ? 'Sending...' : 'Send Verification Code'}
            </button>
          </div>
        ) : (
          // Step 2: Enter OTP
          <form onSubmit={handleVerify}>
            <p className="text-gray-600 text-center mb-6">
              Enter the 6-digit code sent to
              <span className="block font-medium">{maskedEmail}</span>
            </p>

            <OtpInput
              value={code}
              onChange={setCode}
              length={6}
              disabled={verifyingOtp}
            />

            {error && (
              <div className="mt-4 p-3 bg-red-50 border border-red-200 rounded-lg">
                <p className="text-red-700 text-sm">{error}</p>
                {attemptsRemaining !== null && attemptsRemaining > 0 && (
                  <p className="text-red-600 text-xs mt-1">
                    {attemptsRemaining} attempt{attemptsRemaining !== 1 ? 's' : ''} remaining
                  </p>
                )}
              </div>
            )}

            <button
              type="submit"
              disabled={code.length !== 6 || verifyingOtp}
              className="w-full btn-primary py-3 mt-6"
            >
              {verifyingOtp ? 'Verifying...' : 'Verify Code'}
            </button>

            <button
              type="button"
              onClick={handleSendOtp}
              disabled={sendingOtp}
              className="w-full text-blue-600 hover:text-blue-700 text-sm mt-4"
            >
              {sendingOtp ? 'Sending...' : 'Resend Code'}
            </button>
          </form>
        )}

        {/* Expiration notice */}
        {otpSent && (
          <p className="text-gray-500 text-xs text-center mt-6">
            Code expires in 10 minutes
          </p>
        )}
      </div>
    </div>
  );
}
```

### OTP Input Component

```typescript
interface OtpInputProps {
  value: string;
  onChange: (value: string) => void;
  length?: number;
  disabled?: boolean;
}

function OtpInput({ value, onChange, length = 6, disabled }: OtpInputProps) {
  const inputRefs = useRef<(HTMLInputElement | null)[]>([]);

  const handleChange = (index: number, char: string) => {
    if (!/^\d*$/.test(char)) return; // Only digits

    const newValue = value.split('');
    newValue[index] = char;
    const result = newValue.join('').slice(0, length);
    onChange(result);

    // Auto-focus next input
    if (char && index < length - 1) {
      inputRefs.current[index + 1]?.focus();
    }
  };

  const handleKeyDown = (index: number, e: React.KeyboardEvent) => {
    if (e.key === 'Backspace' && !value[index] && index > 0) {
      inputRefs.current[index - 1]?.focus();
    }
  };

  const handlePaste = (e: React.ClipboardEvent) => {
    e.preventDefault();
    const pasted = e.clipboardData.getData('text').replace(/\D/g, '').slice(0, length);
    onChange(pasted);

    // Focus last filled or next empty
    const focusIndex = Math.min(pasted.length, length - 1);
    inputRefs.current[focusIndex]?.focus();
  };

  return (
    <div className="flex gap-2 justify-center">
      {Array.from({ length }).map((_, index) => (
        <input
          key={index}
          ref={(el) => { inputRefs.current[index] = el; }}
          type="text"
          inputMode="numeric"
          maxLength={1}
          value={value[index] || ''}
          onChange={(e) => handleChange(index, e.target.value)}
          onKeyDown={(e) => handleKeyDown(index, e)}
          onPaste={handlePaste}
          disabled={disabled}
          className="w-12 h-14 text-center text-2xl font-bold border-2 border-gray-300
                     rounded-lg focus:border-blue-500 focus:ring-2 focus:ring-blue-200
                     disabled:bg-gray-100 disabled:cursor-not-allowed"
        />
      ))}
    </div>
  );
}
```

---

## Status Screens

### Completed Screen

```typescript
function CompletedScreen({ completedAt }: { completedAt: string | null }) {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50 p-4">
      <div className="bg-white rounded-lg shadow-lg max-w-md w-full p-8 text-center">
        <div className="w-20 h-20 bg-green-100 rounded-full flex items-center justify-center mx-auto mb-6">
          <CheckIcon className="w-10 h-10 text-green-600" />
        </div>
        <h1 className="text-2xl font-bold text-gray-900 mb-2">
          Form Submitted
        </h1>
        <p className="text-gray-600">
          Thank you! Your response has been recorded.
        </p>
        {completedAt && (
          <p className="text-gray-500 text-sm mt-4">
            Submitted on {formatDate(completedAt)}
          </p>
        )}
      </div>
    </div>
  );
}
```

### Expired Screen

```typescript
function ExpiredScreen() {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50 p-4">
      <div className="bg-white rounded-lg shadow-lg max-w-md w-full p-8 text-center">
        <div className="w-20 h-20 bg-yellow-100 rounded-full flex items-center justify-center mx-auto mb-6">
          <ClockIcon className="w-10 h-10 text-yellow-600" />
        </div>
        <h1 className="text-2xl font-bold text-gray-900 mb-2">
          Link Expired
        </h1>
        <p className="text-gray-600">
          This form link has expired. Please contact the sender to request a new link.
        </p>
      </div>
    </div>
  );
}
```

### Error Card

```typescript
function ErrorCard({ error }: { error: unknown }) {
  const getErrorMessage = () => {
    if (typeof error === 'object' && error !== null) {
      const err = error as { status?: number; message?: string };

      switch (err.status) {
        case 401:
        case 403:
          return 'This link is invalid or has expired.';
        case 404:
          return 'Form not found. The link may be incorrect.';
        default:
          return err.message || 'An unexpected error occurred.';
      }
    }
    return 'An unexpected error occurred.';
  };

  return (
    <div className="bg-white rounded-lg shadow-lg max-w-md w-full p-8 text-center">
      <div className="w-20 h-20 bg-red-100 rounded-full flex items-center justify-center mx-auto mb-6">
        <XIcon className="w-10 h-10 text-red-600" />
      </div>
      <h1 className="text-2xl font-bold text-gray-900 mb-2">
        Unable to Load Form
      </h1>
      <p className="text-gray-600">{getErrorMessage()}</p>
    </div>
  );
}
```

---

## Next.js Integration

### App Router Page

```typescript
// app/[urlId]/forms/public/page.tsx
import { PublicFormPage } from '@/components/PublicFormPage';

interface PageProps {
  params: { urlId: string };
  searchParams: { token?: string };
}

export default function Page({ params, searchParams }: PageProps) {
  const { urlId } = params;
  const { token } = searchParams;

  if (!token) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <ErrorCard error={{ status: 400, message: 'Missing token parameter' }} />
      </div>
    );
  }

  return <PublicFormPage urlId={urlId} token={token} />;
}

// Disable caching for this page
export const dynamic = 'force-dynamic';
```

### Middleware for Token Validation

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const { pathname, searchParams } = request.nextUrl;

  // Check if this is a public form route
  if (pathname.match(/^\/[^\/]+\/forms\/public$/)) {
    const token = searchParams.get('token');

    if (!token) {
      return NextResponse.redirect(
        new URL('/error?code=missing_token', request.url)
      );
    }

    // Basic token format validation
    if (token.length < 20) {
      return NextResponse.redirect(
        new URL('/error?code=invalid_token', request.url)
      );
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: '/:urlId/forms/public',
};
```

---

## Utility Functions

```typescript
// Format date for display
function formatDate(dateString: string | null): string {
  if (!dateString) return '';

  return new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
  }).format(new Date(dateString));
}

// Mask email for privacy
function maskEmail(email: string): string {
  const [local, domain] = email.split('@');
  const [domainName, ...rest] = domain.split('.');

  const maskPart = (str: string) => {
    if (str.length <= 2) return str[0] + '*';
    return str[0] + '*'.repeat(str.length - 2) + str[str.length - 1];
  };

  return `${maskPart(local)}@${maskPart(domainName)}.${rest.join('.')}`;
}

// Check if form is expired
function isExpired(expiresAt: string | null): boolean {
  if (!expiresAt) return false;
  return new Date(expiresAt) < new Date();
}

// Build magic link URL
function buildMagicLink(
  baseUrl: string,
  urlId: string,
  token: string
): string {
  return `${baseUrl}/${urlId}/forms/public?token=${token}`;
}
```

---

## Testing Utilities

```typescript
// Mock client for testing
class MockPublicFormsClient extends PublicFormsClient {
  private mockResponses: Map<string, unknown> = new Map();

  setMockResponse(method: string, response: unknown) {
    this.mockResponses.set(method, response);
  }

  async getForm(): Promise<PublicFormResponse> {
    if (this.mockResponses.has('getForm')) {
      return this.mockResponses.get('getForm') as PublicFormResponse;
    }
    throw new Error('Mock not configured');
  }

  async sendOtp(): Promise<OtpSendResponse> {
    if (this.mockResponses.has('sendOtp')) {
      return this.mockResponses.get('sendOtp') as OtpSendResponse;
    }
    return {
      message: 'Verification code sent',
      masked_email: 't***t@e***.com',
      expires_in: 600,
    };
  }

  async verifyOtp(): Promise<OtpVerifySuccessResponse> {
    if (this.mockResponses.has('verifyOtp')) {
      return this.mockResponses.get('verifyOtp') as OtpVerifySuccessResponse;
    }
    throw { code: 'INVALID_OTP', attempts_remaining: 4 };
  }
}

// Test helper
function renderPublicForm(urlId: string, token: string) {
  return render(
    <PublicFormPage urlId={urlId} token={token} />
  );
}
```
