---
name: forms-schemas-sdk
description: Use 23blocks Forms SDK in TypeScript and React. Use when building frontend forms, handling form state, or implementing form UI components.
allowed-tools: Read, Write, Grep, Glob
user-invocable: true
---

# Forms Schemas SDK

TypeScript and React SDK for 23blocks form schemas.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Installation

```bash
npm install @23blocks/forms @23blocks/forms-react
```

---

## TypeScript Client

### Initialize Client

```typescript
import { FormsClient } from '@23blocks/forms';

// CRITICAL: Verify env vars before initializing
if (!process.env.BLOCKS_API_URL || !process.env.BLOCKS_AUTH_TOKEN || !process.env.BLOCKS_API_KEY) {
  throw new Error('Missing required env vars: BLOCKS_API_URL, BLOCKS_AUTH_TOKEN, BLOCKS_API_KEY');
}

const forms = new FormsClient({
  apiKey: process.env.BLOCKS_API_KEY,
  authToken: process.env.BLOCKS_AUTH_TOKEN,
  baseUrl: process.env.BLOCKS_API_URL,  // e.g., https://forms.api.us.23blocks.com
  timeout: 30000, // optional, ms
});
```

### Schema Operations

#### Create Schema

```typescript
// forms.schemas.create(definition: SchemaDefinition): Promise<Schema>

const schema = await forms.schemas.create({
  name: 'contact_form',
  description: 'Contact us form',
  fields: [
    {
      name: 'email',
      type: 'email',
      label: 'Email Address',
      required: true,
      validation: { format: 'email' }
    },
    {
      name: 'message',
      type: 'textarea',
      label: 'Message',
      required: true,
      validation: { minLength: 10, maxLength: 2000 }
    }
  ],
  settings: {
    submitLabel: 'Send Message',
    successMessage: 'Thanks for reaching out!'
  }
});

console.log(schema.id); // frm_abc123
```

#### Get Schema

```typescript
// forms.schemas.get(id: string): Promise<Schema>

const schema = await forms.schemas.get('frm_abc123');
console.log(schema.fields);
```

#### Update Schema

```typescript
// forms.schemas.update(id: string, updates: Partial<SchemaDefinition>): Promise<Schema>

const updated = await forms.schemas.update('frm_abc123', {
  description: 'Updated contact form',
  fields: [
    // ... updated fields
  ]
});

console.log(updated.version); // 2
```

#### Delete Schema

```typescript
// forms.schemas.delete(id: string): Promise<void>

await forms.schemas.delete('frm_abc123');
```

#### List Schemas

```typescript
// forms.schemas.list(options?: ListOptions): Promise<PaginatedResponse<Schema>>

const { data, pagination } = await forms.schemas.list({
  page: 1,
  limit: 20,
  status: 'active',
  search: 'contact'
});

console.log(`Found ${pagination.total} schemas`);
```

### Error Handling

```typescript
import { FormsError } from '@23blocks/forms';

try {
  await forms.schemas.create(invalidSchema);
} catch (error) {
  if (error instanceof FormsError) {
    console.error(`${error.code}: ${error.message}`);
    // INVALID_SCHEMA: Schema validation failed

    if (error.details) {
      console.error(error.details);
      // { "fields[0].type": "Invalid field type" }
    }
  }
}
```

---

## React SDK

### FormsProvider

Wrap your app to provide forms context:

```tsx
import { FormsProvider } from '@23blocks/forms-react';

function App() {
  return (
    <FormsProvider apiKey={process.env.REACT_APP_API_KEY}>
      <YourApp />
    </FormsProvider>
  );
}
```

### useSchema Hook

Load and access a form schema:

```tsx
import { useSchema } from '@23blocks/forms-react';

// useSchema(schemaId: string): UseSchemaResult

function FormPage({ schemaId }: { schemaId: string }) {
  const { schema, isLoading, error } = useSchema(schemaId);

  if (isLoading) return <div>Loading form...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <h1>{schema.name}</h1>
      <p>{schema.description}</p>
      {/* Render form fields */}
    </div>
  );
}
```

**Returns:**
- `schema: Schema | null` - The loaded schema
- `isLoading: boolean` - Loading state
- `error: FormsError | null` - Error if load failed
- `refetch: () => Promise<void>` - Reload schema

### useForm Hook

Manage form state and validation:

```tsx
import { useForm } from '@23blocks/forms-react';

// useForm(schemaId: string, options?: UseFormOptions): UseFormResult

function ContactForm() {
  const {
    fields,
    values,
    errors,
    touched,
    isValid,
    handleChange,
    handleBlur,
    setFieldValue,
    reset
  } = useForm('frm_abc123', {
    initialValues: { email: '', message: '' },
    validateOnChange: true,
    validateOnBlur: true
  });

  return (
    <form>
      {fields.map(field => (
        <div key={field.name}>
          <label>{field.label}</label>
          <input
            name={field.name}
            type={field.type}
            value={values[field.name] || ''}
            onChange={handleChange}
            onBlur={handleBlur}
            placeholder={field.placeholder}
          />
          {touched[field.name] && errors[field.name] && (
            <span className="error">{errors[field.name]}</span>
          )}
        </div>
      ))}
    </form>
  );
}
```

**Options:**
- `initialValues: Record<string, any>` - Default field values
- `validateOnChange: boolean` - Validate when values change (default: true)
- `validateOnBlur: boolean` - Validate on field blur (default: true)
- `validateOnMount: boolean` - Validate on mount (default: false)

**Returns:**
- `fields: Field[]` - Schema field definitions
- `values: Record<string, any>` - Current form values
- `errors: Record<string, string>` - Validation errors
- `touched: Record<string, boolean>` - Fields that have been touched
- `isValid: boolean` - Whether form passes validation
- `isDirty: boolean` - Whether form has been modified
- `handleChange: (e: ChangeEvent) => void` - Change handler
- `handleBlur: (e: FocusEvent) => void` - Blur handler
- `setFieldValue: (name: string, value: any) => void` - Set field programmatically
- `setFieldError: (name: string, error: string) => void` - Set error manually
- `reset: () => void` - Reset to initial values
- `validate: () => Promise<boolean>` - Trigger full validation

### useFormSubmit Hook

Handle form submission:

```tsx
import { useForm, useFormSubmit } from '@23blocks/forms-react';

// useFormSubmit(schemaId: string): UseFormSubmitResult

function ContactForm() {
  const { values, errors, isValid, handleChange } = useForm('frm_abc123');
  const { submit, isSubmitting, submitError, isSuccess } = useFormSubmit('frm_abc123');

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    if (!isValid) return;

    await submit(values);
  };

  if (isSuccess) {
    return <div>Thanks for your message!</div>;
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}

      {submitError && (
        <div className="error">{submitError.message}</div>
      )}

      <button type="submit" disabled={!isValid || isSubmitting}>
        {isSubmitting ? 'Sending...' : 'Send Message'}
      </button>
    </form>
  );
}
```

**Returns:**
- `submit: (values: Record<string, any>) => Promise<Submission>` - Submit function
- `isSubmitting: boolean` - Submission in progress
- `submitError: FormsError | null` - Submission error
- `isSuccess: boolean` - Submission succeeded
- `submission: Submission | null` - Submission result
- `reset: () => void` - Reset submission state

### FormField Component

Pre-built field components:

```tsx
import { FormField, Form } from '@23blocks/forms-react';

function ContactForm() {
  return (
    <Form schemaId="frm_abc123" onSuccess={() => alert('Submitted!')}>
      <FormField name="email" />
      <FormField name="subject" />
      <FormField name="message" />
      <button type="submit">Send</button>
    </Form>
  );
}
```

**FormField Props:**
- `name: string` - Field name (required)
- `className: string` - CSS class
- `labelClassName: string` - Label CSS class
- `inputClassName: string` - Input CSS class
- `errorClassName: string` - Error CSS class
- `hideLabel: boolean` - Hide the label
- `hideError: boolean` - Hide error message
- `render: (props: FieldRenderProps) => ReactNode` - Custom render function

---

## TypeScript Types

```typescript
// Exported from @23blocks/forms

export interface Schema {
  id: string;
  name: string;
  description: string;
  fields: Field[];
  settings: SchemaSettings;
  version: number;
  status: 'active' | 'archived';
  created_at: string;
  updated_at: string;
}

export interface Field {
  name: string;
  type: FieldType;
  label: string;
  required: boolean;
  placeholder?: string;
  default?: any;
  options?: Option[];
  validation?: Validation;
  conditions?: Condition[];
}

export type FieldType =
  | 'text'
  | 'email'
  | 'tel'
  | 'number'
  | 'select'
  | 'multiselect'
  | 'checkbox'
  | 'textarea'
  | 'date'
  | 'datetime'
  | 'file'
  | 'image';

export interface Validation {
  format?: 'email' | 'url' | 'phone' | 'uuid';
  pattern?: string;
  minLength?: number;
  maxLength?: number;
  min?: number;
  max?: number;
  unique?: boolean;
  custom?: string;
}

export interface Option {
  value: string;
  label: string;
}

export interface Submission {
  id: string;
  schema_id: string;
  data: Record<string, any>;
  created_at: string;
}

export class FormsError extends Error {
  code: string;
  message: string;
  details?: Record<string, string>;
}
```
