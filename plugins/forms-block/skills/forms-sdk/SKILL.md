---
name: forms-sdk
description: TypeScript SDK for 23blocks Forms API. Use for type definitions, API client setup, and React integration patterns.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Forms SDK (TypeScript/React)

TypeScript types, API client, and React patterns for 23blocks Forms API integration.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Installation

```bash
npm install @23blocks/forms-sdk
# or
yarn add @23blocks/forms-sdk
```

## API Client Setup

### Basic Configuration

```typescript
import { FormsClient } from '@23blocks/forms-sdk';

// CRITICAL: Verify env vars before initializing
if (!process.env.BLOCKS_API_URL || !process.env.BLOCKS_AUTH_TOKEN || !process.env.BLOCKS_API_KEY) {
  throw new Error('Missing required env vars: BLOCKS_API_URL, BLOCKS_AUTH_TOKEN, BLOCKS_API_KEY');
}

const client = new FormsClient({
  baseUrl: process.env.BLOCKS_API_URL,  // e.g., https://forms.api.us.23blocks.com
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

  // CRITICAL: Verify env var before initializing
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

---

## Type Definitions

### Form Types

```typescript
// Form type enum
type FormType =
  | 'landing'
  | 'survey'
  | 'appointment'
  | 'subscription'
  | 'referral'
  | 'app_form';

// Base form interface
interface Form {
  id: string;
  type: 'form';
  attributes: FormAttributes;
  relationships?: FormRelationships;
}

interface FormAttributes {
  unique_id: string;
  name: string;
  description: string | null;
  form_type: FormType;
  status: 'active' | 'inactive' | 'archived';
  settings: FormSettings;
  form_schema_id: string | null; // For app_form type
  created_at: string;
  updated_at: string;
}

interface FormSettings {
  allow_multiple_submissions: boolean;
  require_authentication: boolean;
  notification_email: string | null;
  redirect_url: string | null;
  confirmation_message: string | null;
  [key: string]: unknown;
}

interface FormRelationships {
  form_schema?: {
    data: { id: string; type: 'form_schema' } | null;
  };
}
```

### API Response Types

```typescript
// JSON:API response wrapper
interface ApiResponse<T> {
  data: T;
  included?: Array<FormSchema | FormSchemaVersion>;
  meta?: PaginationMeta;
}

interface ApiListResponse<T> {
  data: T[];
  included?: Array<FormSchema | FormSchemaVersion>;
  meta: PaginationMeta;
}

interface PaginationMeta {
  totalPages: number;
  totalRecords: number;
  currentPage?: number;
  perPage?: number;
}

// Error response
interface ApiError {
  status: number;
  code: string;
  title: string;
  detail: string;
}

interface ApiErrorResponse {
  errors: ApiError[];
}
```

### Form Instance Types

```typescript
// Landing form submission
interface LandingInstance {
  id: string;
  type: 'landing_instance';
  attributes: {
    unique_id: string;
    first_name: string | null;
    last_name: string | null;
    email: string;
    phone: string | null;
    company: string | null;
    message: string | null;
    source: string | null;
    utm_source: string | null;
    utm_medium: string | null;
    utm_campaign: string | null;
    payload: Record<string, unknown>;
    created_at: string;
  };
}

// Survey response
interface SurveyInstance {
  id: string;
  type: 'survey_instance';
  attributes: {
    unique_id: string;
    user_unique_id: string | null;
    assigned_to_email: string;
    assigned_to_name: string | null;
    status: 'pending' | 'in_progress' | 'completed' | 'expired';
    responses: unknown[];
    magic_link_token: string;
    expires_at: string | null;
    started_at: string | null;
    completed_at: string | null;
    created_at: string;
  };
}

// Appointment booking
interface Appointment {
  id: string;
  type: 'appointment';
  attributes: {
    unique_id: string;
    first_name: string;
    last_name: string;
    email: string;
    phone: string | null;
    appointment_date: string; // YYYY-MM-DD
    appointment_time: string; // HH:MM
    status: 'pending' | 'confirmed' | 'cancelled';
    notes: string | null;
    metadata: Record<string, unknown>;
    confirmation_token: string;
    cancellation_token: string;
    confirmed_at: string | null;
    cancelled_at: string | null;
    created_at: string;
  };
}

// Newsletter subscription
interface Subscription {
  id: string;
  type: 'subscription';
  attributes: {
    unique_id: string;
    email: string;
    first_name: string | null;
    last_name: string | null;
    status: 'active' | 'unsubscribed';
    source: string | null;
    preferences: Record<string, boolean>;
    subscribed_at: string;
    unsubscribed_at: string | null;
    created_at: string;
  };
}

// Referral tracking
interface Referral {
  id: string;
  type: 'referral';
  attributes: {
    unique_id: string;
    referrer_email: string;
    referrer_name: string | null;
    referred_email: string;
    referred_name: string | null;
    status: 'pending' | 'converted' | 'expired';
    referral_code: string | null;
    reward_status: 'pending' | 'paid' | 'cancelled';
    reward_amount: number | null;
    metadata: Record<string, unknown>;
    converted_at: string | null;
    created_at: string;
  };
}
```

---

## React Hooks

### useForm Hook

```typescript
import { useState, useEffect } from 'react';
import { FormsClient, Form, ApiError } from '@23blocks/forms-sdk';

interface UseFormOptions {
  client: FormsClient;
  formId: string;
}

interface UseFormResult {
  form: Form | null;
  loading: boolean;
  error: ApiError | null;
  refetch: () => Promise<void>;
}

function useForm({ client, formId }: UseFormOptions): UseFormResult {
  const [form, setForm] = useState<Form | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<ApiError | null>(null);

  const fetchForm = async () => {
    setLoading(true);
    setError(null);
    try {
      const response = await client.forms.get(formId);
      setForm(response.data);
    } catch (err) {
      setError(err as ApiError);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchForm();
  }, [formId]);

  return { form, loading, error, refetch: fetchForm };
}
```

### useForms Hook (List)

```typescript
interface UseFormsOptions {
  client: FormsClient;
  page?: number;
  perPage?: number;
  formType?: FormType;
}

interface UseFormsResult {
  forms: Form[];
  loading: boolean;
  error: ApiError | null;
  pagination: PaginationMeta | null;
  refetch: () => Promise<void>;
}

function useForms({
  client,
  page = 1,
  perPage = 20,
  formType
}: UseFormsOptions): UseFormsResult {
  const [forms, setForms] = useState<Form[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<ApiError | null>(null);
  const [pagination, setPagination] = useState<PaginationMeta | null>(null);

  const fetchForms = async () => {
    setLoading(true);
    setError(null);
    try {
      const response = await client.forms.list({
        page,
        records: perPage,
        form_type: formType,
      });
      setForms(response.data);
      setPagination(response.meta);
    } catch (err) {
      setError(err as ApiError);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchForms();
  }, [page, perPage, formType]);

  return { forms, loading, error, pagination, refetch: fetchForms };
}
```

### useCreateForm Hook

```typescript
interface UseCreateFormOptions {
  client: FormsClient;
  onSuccess?: (form: Form) => void;
  onError?: (error: ApiError) => void;
}

interface CreateFormInput {
  name: string;
  description?: string;
  form_type: FormType;
  settings?: Partial<FormSettings>;
  form_schema_id?: string;
}

interface UseCreateFormResult {
  createForm: (input: CreateFormInput) => Promise<Form>;
  loading: boolean;
  error: ApiError | null;
}

function useCreateForm({
  client,
  onSuccess,
  onError
}: UseCreateFormOptions): UseCreateFormResult {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<ApiError | null>(null);

  const createForm = async (input: CreateFormInput): Promise<Form> => {
    setLoading(true);
    setError(null);
    try {
      const response = await client.forms.create({ form: input });
      onSuccess?.(response.data);
      return response.data;
    } catch (err) {
      const apiError = err as ApiError;
      setError(apiError);
      onError?.(apiError);
      throw apiError;
    } finally {
      setLoading(false);
    }
  };

  return { createForm, loading, error };
}
```

---

## Client Methods

### Forms CRUD

```typescript
const client = new FormsClient({ /* config */ });

// List forms
const { data: forms, meta } = await client.forms.list({
  page: 1,
  records: 20,
  form_type: 'survey',
});

// Get single form
const { data: form } = await client.forms.get('form-uuid');

// Create form
const { data: newForm } = await client.forms.create({
  form: {
    name: 'Customer Feedback',
    form_type: 'survey',
    description: 'Post-purchase survey',
    settings: {
      allow_multiple_submissions: false,
      notification_email: 'team@company.com',
    },
  },
});

// Update form
const { data: updated } = await client.forms.update('form-uuid', {
  form: {
    name: 'Updated Name',
    status: 'active',
  },
});

// Delete form
await client.forms.delete('form-uuid');
```

### Landing Instances

```typescript
// Create landing submission
const { data: submission } = await client.landings.create('form-uuid', {
  landing_instance: {
    first_name: 'Jane',
    last_name: 'Smith',
    email: 'jane@example.com',
    company: 'Acme Corp',
    message: 'Interested in your services',
    utm_source: 'google',
    utm_campaign: 'brand',
    payload: {
      product_interest: 'Enterprise',
      company_size: '50-100',
    },
  },
});

// List submissions
const { data: submissions } = await client.landings.list('form-uuid', {
  page: 1,
  records: 50,
});
```

### Appointments

```typescript
// Create appointment
const { data: appointment } = await client.appointments.create('form-uuid', {
  appointment: {
    first_name: 'John',
    last_name: 'Doe',
    email: 'john@example.com',
    appointment_date: '2025-02-15',
    appointment_time: '14:30',
    notes: 'First consultation',
    metadata: {
      service: 'Consultation',
      provider_id: 'dr-smith',
    },
  },
});

// Confirm appointment
await client.appointments.confirm('form-uuid', 'apt-uuid');

// Cancel appointment
await client.appointments.cancel('form-uuid', 'apt-uuid', {
  reason: 'Patient requested cancellation',
});
```

---

## Error Handling

```typescript
import { FormsClient, isApiError, ApiErrorResponse } from '@23blocks/forms-sdk';

async function createFormWithErrorHandling() {
  const client = new FormsClient({ /* config */ });

  try {
    const { data } = await client.forms.create({
      form: { name: 'Test', form_type: 'landing' },
    });
    return data;
  } catch (error) {
    if (isApiError(error)) {
      const apiError = error as ApiErrorResponse;

      switch (apiError.errors[0]?.status) {
        case 401:
          console.error('Authentication failed');
          break;
        case 403:
          console.error('Permission denied');
          break;
        case 422:
          console.error('Validation error:', apiError.errors[0].detail);
          break;
        default:
          console.error('API error:', apiError.errors[0].title);
      }
    }
    throw error;
  }
}
```

---

## React Query Integration

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { FormsClient, Form, CreateFormInput } from '@23blocks/forms-sdk';

const client = new FormsClient({ /* config */ });

// Query hook
function useFormsQuery(formType?: FormType) {
  return useQuery({
    queryKey: ['forms', formType],
    queryFn: () => client.forms.list({ form_type: formType }),
    select: (response) => response.data,
  });
}

// Mutation hook
function useCreateFormMutation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (input: CreateFormInput) =>
      client.forms.create({ form: input }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['forms'] });
    },
  });
}

// Usage
function FormsPage() {
  const { data: forms, isLoading } = useFormsQuery('survey');
  const createMutation = useCreateFormMutation();

  const handleCreate = () => {
    createMutation.mutate({
      name: 'New Survey',
      form_type: 'survey',
    });
  };

  if (isLoading) return <Loading />;

  return (
    <div>
      <button onClick={handleCreate} disabled={createMutation.isPending}>
        Create Survey
      </button>
      <FormsList forms={forms} />
    </div>
  );
}
```

---

## Constants & Utilities

```typescript
// Form type labels
export const FORM_TYPE_LABELS: Record<FormType, string> = {
  landing: 'Landing Page',
  survey: 'Survey',
  appointment: 'Appointment',
  subscription: 'Subscription',
  referral: 'Referral',
  app_form: 'Application Form',
};

// Status colors (Tailwind)
export const STATUS_COLORS = {
  active: 'bg-green-100 text-green-800',
  inactive: 'bg-gray-100 text-gray-800',
  archived: 'bg-red-100 text-red-800',
  pending: 'bg-yellow-100 text-yellow-800',
  completed: 'bg-blue-100 text-blue-800',
  confirmed: 'bg-green-100 text-green-800',
  cancelled: 'bg-red-100 text-red-800',
};

// Date formatting
export function formatFormDate(dateString: string): string {
  return new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
  }).format(new Date(dateString));
}
```
