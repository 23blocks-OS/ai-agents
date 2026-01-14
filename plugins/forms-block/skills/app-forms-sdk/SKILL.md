---
name: app-forms-sdk
description: TypeScript SDK for 23blocks App Forms with schemas, versions, validation rules, and scoring engines. Use for clinical assessments, evaluations, and dynamic business forms.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# App Forms SDK (TypeScript/React)

TypeScript types, React components, and patterns for dynamic business forms with validation and scoring.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

**CRITICAL:** Always verify environment variables before initializing the client:

```typescript
// Pre-flight check
if (!process.env.BLOCKS_API_URL || !process.env.BLOCKS_AUTH_TOKEN || !process.env.BLOCKS_API_KEY) {
  throw new Error('Missing required env vars: BLOCKS_API_URL, BLOCKS_AUTH_TOKEN, BLOCKS_API_KEY');
}
```

## Overview

App Forms are structured business forms with:
- **Dynamic Schemas**: Define form structure with embedded business rules
- **Version Control**: Lock instances to specific schema versions
- **Validation Engine**: Required fields, patterns, ranges, custom validators
- **Scoring Engine**: Sum, average, range mapping, conditional logic
- **Assignment Lifecycle**: Pending → In Progress → Completed

**Use Cases:**
- Clinical assessments (GAD-7, PHQ-9)
- Employee evaluations
- Customer intake forms
- Quality control checklists

---

## Type Definitions

### Form Schema Types

```typescript
// Form schema (template)
interface FormSchema {
  id: string;
  type: 'form_schema';
  attributes: {
    unique_id: string;
    name: string;
    description: string | null;
    created_at: string;
    updated_at: string;
  };
  relationships?: {
    versions: {
      data: Array<{ id: string; type: 'form_schema_version' }>;
    };
    forms: {
      data: Array<{ id: string; type: 'form' }>;
    };
  };
}

// Schema version (immutable once published)
interface FormSchemaVersion {
  id: string;
  type: 'form_schema_version';
  attributes: {
    unique_id: string;
    version_number: number;
    status: 'draft' | 'published' | 'archived';
    form_fields: FormFieldsConfig;
    published_at: string | null;
    created_at: string;
    updated_at: string;
  };
}

// Form fields configuration
interface FormFieldsConfig {
  fields: FormField[];
  validationRules?: ValidationRules;
  scoringRules?: ScoringRules;
}
```

### Field Types

```typescript
type FieldType =
  | 'text'
  | 'textarea'
  | 'number'
  | 'email'
  | 'phone'
  | 'date'
  | 'time'
  | 'select'
  | 'radio'
  | 'checkbox'
  | 'display'    // Read-only display text
  | 'signature'
  | 'file';

interface FormField {
  id: string;
  type: FieldType;
  label: string;
  description?: string;
  placeholder?: string;
  required?: boolean;
  options?: FieldOption[];      // For select, radio, checkbox
  scores?: number[];            // Parallel to options for scoring
  defaultValue?: unknown;
  validation?: FieldValidation;
  conditional?: ConditionalLogic;
}

interface FieldOption {
  value: string;
  label: string;
}

interface FieldValidation {
  minLength?: number;
  maxLength?: number;
  min?: number;
  max?: number;
  pattern?: string;
  patternMessage?: string;
}

interface ConditionalLogic {
  dependsOn: string;           // Field ID
  showWhen: {
    operator: 'equals' | 'not_equals' | 'greater_than' | 'less_than' | 'contains';
    value: unknown;
  };
}
```

### Validation Rules

```typescript
interface ValidationRules {
  required_fields?: string[];           // Field IDs that must have values
  min_length?: Record<string, number>;  // { fieldId: minLength }
  max_length?: Record<string, number>;  // { fieldId: maxLength }
  pattern?: Record<string, PatternRule>;
  range?: Record<string, RangeRule>;
  custom?: CustomValidator[];
}

interface PatternRule {
  regex: string;
  message: string;
}

interface RangeRule {
  min?: number;
  max?: number;
  message?: string;
}

interface CustomValidator {
  name: string;
  fields: string[];
  validator: 'email' | 'phone' | 'url' | 'date_future' | 'date_past';
  message?: string;
}
```

### Scoring Rules

```typescript
interface ScoringRules {
  sum?: SumRule[];
  average?: AverageRule[];
  count?: CountRule[];
  range?: RangeRule[];
  conditional?: ConditionalRule[];
  custom?: CustomScoringRule[];
}

interface SumRule {
  name: string;
  fields: string[];              // Field IDs to sum
  output: string;                // Output key in calculated_scores
}

interface AverageRule {
  name: string;
  fields: string[];
  output: string;
  precision?: number;
}

interface CountRule {
  name: string;
  fields: string[];
  countValue: unknown;           // Value to count occurrences of
  output: string;
}

interface RangeMappingRule {
  name: string;
  input: string;                 // Score key to map
  output: string;
  ranges: Array<{
    min: number;
    max: number;
    label: string;
  }>;
}

interface ConditionalRule {
  name: string;
  condition: {
    field: string;
    operator: 'equals' | 'greater_than' | 'less_than' | 'between';
    value: unknown;
  };
  then: { output: string; value: unknown };
  else?: { output: string; value: unknown };
}

interface CustomScoringRule {
  name: string;
  calculator: 'percentage_complete' | 'weighted_average';
  config: Record<string, unknown>;
  output: string;
}
```

### App Form Instance

```typescript
interface AppFormInstance {
  id: string;
  type: 'app_form_instance';
  attributes: {
    unique_id: string;
    user_unique_id: string | null;
    assigned_to_email: string;
    assigned_to_name: string | null;
    assigned_by_email: string | null;
    assigned_by_name: string | null;
    status: AppFormStatus;
    responses: Array<unknown>;        // Array indexed by field order
    calculated_scores: Record<string, unknown>;
    metadata: Record<string, unknown>;
    magic_link_token: string;
    expires_at: string | null;
    started_at: string | null;
    completed_at: string | null;
    created_at: string;
    updated_at: string;
  };
  relationships?: {
    form: { data: { id: string; type: 'form' } };
    form_schema_version: { data: { id: string; type: 'form_schema_version' } };
  };
}

type AppFormStatus = 'pending' | 'in_progress' | 'completed' | 'cancelled' | 'expired';
```

---

## React Components

### AppFormRenderer Component

```typescript
import React, { useState, useCallback } from 'react';

interface AppFormRendererProps {
  schema: FormSchemaVersion;
  instance: AppFormInstance;
  onSave: (responses: unknown[]) => Promise<void>;
  onSubmit: (responses: unknown[]) => Promise<void>;
  readOnly?: boolean;
}

function AppFormRenderer({
  schema,
  instance,
  onSave,
  onSubmit,
  readOnly = false,
}: AppFormRendererProps) {
  const [responses, setResponses] = useState<unknown[]>(
    instance.attributes.responses || []
  );
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [saving, setSaving] = useState(false);

  const { fields, validationRules } = schema.attributes.form_fields;

  const handleFieldChange = useCallback((index: number, value: unknown) => {
    setResponses(prev => {
      const updated = [...prev];
      updated[index] = value;
      return updated;
    });
    // Clear error on change
    setErrors(prev => {
      const { [fields[index].id]: _, ...rest } = prev;
      return rest;
    });
  }, [fields]);

  const validate = useCallback((): boolean => {
    const newErrors: Record<string, string> = {};

    fields.forEach((field, index) => {
      const value = responses[index];

      // Required check
      if (validationRules?.required_fields?.includes(field.id)) {
        if (value === null || value === undefined || value === '') {
          newErrors[field.id] = `${field.label} is required`;
        }
      }

      // Min/max length
      if (typeof value === 'string') {
        const minLen = validationRules?.min_length?.[field.id];
        const maxLen = validationRules?.max_length?.[field.id];

        if (minLen && value.length < minLen) {
          newErrors[field.id] = `Minimum ${minLen} characters required`;
        }
        if (maxLen && value.length > maxLen) {
          newErrors[field.id] = `Maximum ${maxLen} characters allowed`;
        }
      }

      // Pattern validation
      const pattern = validationRules?.pattern?.[field.id];
      if (pattern && typeof value === 'string') {
        if (!new RegExp(pattern.regex).test(value)) {
          newErrors[field.id] = pattern.message;
        }
      }
    });

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  }, [fields, responses, validationRules]);

  const handleSave = async () => {
    setSaving(true);
    try {
      await onSave(responses);
    } finally {
      setSaving(false);
    }
  };

  const handleSubmit = async () => {
    if (!validate()) return;

    setSaving(true);
    try {
      await onSubmit(responses);
    } finally {
      setSaving(false);
    }
  };

  return (
    <form onSubmit={(e) => { e.preventDefault(); handleSubmit(); }}>
      {fields.map((field, index) => (
        <FormFieldComponent
          key={field.id}
          field={field}
          value={responses[index]}
          onChange={(value) => handleFieldChange(index, value)}
          error={errors[field.id]}
          disabled={readOnly || saving}
        />
      ))}

      {!readOnly && (
        <div className="flex gap-4 mt-6">
          <button
            type="button"
            onClick={handleSave}
            disabled={saving}
            className="btn-secondary"
          >
            Save Draft
          </button>
          <button
            type="submit"
            disabled={saving}
            className="btn-primary"
          >
            Submit
          </button>
        </div>
      )}
    </form>
  );
}
```

### FormFieldComponent

```typescript
interface FormFieldComponentProps {
  field: FormField;
  value: unknown;
  onChange: (value: unknown) => void;
  error?: string;
  disabled?: boolean;
}

function FormFieldComponent({
  field,
  value,
  onChange,
  error,
  disabled,
}: FormFieldComponentProps) {
  // Display field (read-only text)
  if (field.type === 'display') {
    return (
      <div className="mb-4">
        <p className="text-gray-700">{field.label}</p>
        {field.description && (
          <p className="text-sm text-gray-500">{field.description}</p>
        )}
      </div>
    );
  }

  // Radio buttons with scoring
  if (field.type === 'radio' && field.options) {
    return (
      <fieldset className="mb-4">
        <legend className="font-medium">{field.label}</legend>
        {field.description && (
          <p className="text-sm text-gray-500 mb-2">{field.description}</p>
        )}
        <div className="space-y-2">
          {field.options.map((option, optIndex) => (
            <label key={option.value} className="flex items-center gap-2">
              <input
                type="radio"
                name={field.id}
                value={optIndex}
                checked={value === optIndex}
                onChange={() => onChange(optIndex)}
                disabled={disabled}
              />
              <span>{option.label}</span>
            </label>
          ))}
        </div>
        {error && <p className="text-red-500 text-sm mt-1">{error}</p>}
      </fieldset>
    );
  }

  // Text input
  if (field.type === 'text' || field.type === 'email' || field.type === 'phone') {
    return (
      <div className="mb-4">
        <label className="block font-medium mb-1">
          {field.label}
          {field.required && <span className="text-red-500">*</span>}
        </label>
        {field.description && (
          <p className="text-sm text-gray-500 mb-1">{field.description}</p>
        )}
        <input
          type={field.type}
          value={(value as string) || ''}
          onChange={(e) => onChange(e.target.value)}
          placeholder={field.placeholder}
          disabled={disabled}
          className={`input ${error ? 'border-red-500' : ''}`}
        />
        {error && <p className="text-red-500 text-sm mt-1">{error}</p>}
      </div>
    );
  }

  // Textarea
  if (field.type === 'textarea') {
    return (
      <div className="mb-4">
        <label className="block font-medium mb-1">
          {field.label}
          {field.required && <span className="text-red-500">*</span>}
        </label>
        <textarea
          value={(value as string) || ''}
          onChange={(e) => onChange(e.target.value)}
          placeholder={field.placeholder}
          disabled={disabled}
          rows={4}
          className={`textarea ${error ? 'border-red-500' : ''}`}
        />
        {error && <p className="text-red-500 text-sm mt-1">{error}</p>}
      </div>
    );
  }

  // Number input
  if (field.type === 'number') {
    return (
      <div className="mb-4">
        <label className="block font-medium mb-1">
          {field.label}
          {field.required && <span className="text-red-500">*</span>}
        </label>
        <input
          type="number"
          value={(value as number) ?? ''}
          onChange={(e) => onChange(e.target.valueAsNumber || null)}
          min={field.validation?.min}
          max={field.validation?.max}
          disabled={disabled}
          className={`input ${error ? 'border-red-500' : ''}`}
        />
        {error && <p className="text-red-500 text-sm mt-1">{error}</p>}
      </div>
    );
  }

  // Select dropdown
  if (field.type === 'select' && field.options) {
    return (
      <div className="mb-4">
        <label className="block font-medium mb-1">
          {field.label}
          {field.required && <span className="text-red-500">*</span>}
        </label>
        <select
          value={(value as string) || ''}
          onChange={(e) => onChange(e.target.value || null)}
          disabled={disabled}
          className={`select ${error ? 'border-red-500' : ''}`}
        >
          <option value="">Select...</option>
          {field.options.map((option) => (
            <option key={option.value} value={option.value}>
              {option.label}
            </option>
          ))}
        </select>
        {error && <p className="text-red-500 text-sm mt-1">{error}</p>}
      </div>
    );
  }

  return null;
}
```

---

## React Hooks

### useAppFormInstance Hook

```typescript
interface UseAppFormInstanceOptions {
  client: FormsClient;
  formId: string;
  instanceId: string;
}

interface UseAppFormInstanceResult {
  instance: AppFormInstance | null;
  schema: FormSchemaVersion | null;
  loading: boolean;
  error: ApiError | null;
  start: () => Promise<void>;
  saveDraft: (responses: unknown[]) => Promise<void>;
  submit: (responses: unknown[]) => Promise<void>;
  cancel: () => Promise<void>;
}

function useAppFormInstance({
  client,
  formId,
  instanceId,
}: UseAppFormInstanceOptions): UseAppFormInstanceResult {
  const [instance, setInstance] = useState<AppFormInstance | null>(null);
  const [schema, setSchema] = useState<FormSchemaVersion | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<ApiError | null>(null);

  useEffect(() => {
    async function fetchData() {
      setLoading(true);
      try {
        const response = await client.appForms.get(formId, instanceId, {
          include: 'form_schema_version',
        });
        setInstance(response.data);
        // Extract schema from included
        const schemaVersion = response.included?.find(
          (inc) => inc.type === 'form_schema_version'
        );
        setSchema(schemaVersion as FormSchemaVersion);
      } catch (err) {
        setError(err as ApiError);
      } finally {
        setLoading(false);
      }
    }
    fetchData();
  }, [formId, instanceId]);

  const start = async () => {
    const response = await client.appForms.start(formId, instanceId);
    setInstance(response.data);
  };

  const saveDraft = async (responses: unknown[]) => {
    const response = await client.appForms.update(formId, instanceId, {
      app_form_instance: { responses },
    });
    setInstance(response.data);
  };

  const submit = async (responses: unknown[]) => {
    const response = await client.appForms.submit(formId, instanceId, {
      responses,
    });
    setInstance(response.data);
  };

  const cancel = async () => {
    const response = await client.appForms.cancel(formId, instanceId);
    setInstance(response.data);
  };

  return { instance, schema, loading, error, start, saveDraft, submit, cancel };
}
```

### useFormSchema Hook

```typescript
function useFormSchema(client: FormsClient, schemaId: string) {
  const [schema, setSchema] = useState<FormSchema | null>(null);
  const [versions, setVersions] = useState<FormSchemaVersion[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchSchema() {
      setLoading(true);
      try {
        const schemaResponse = await client.formSchemas.get(schemaId);
        setSchema(schemaResponse.data);

        const versionsResponse = await client.formSchemaVersions.list(schemaId);
        setVersions(versionsResponse.data);
      } finally {
        setLoading(false);
      }
    }
    fetchSchema();
  }, [schemaId]);

  const createVersion = async (formFields: FormFieldsConfig) => {
    const response = await client.formSchemaVersions.create(schemaId, {
      form_schema_version: { form_fields: formFields },
    });
    setVersions((prev) => [...prev, response.data]);
    return response.data;
  };

  const publishVersion = async (versionId: string) => {
    const response = await client.formSchemaVersions.publish(schemaId, versionId);
    // Update local state - published version, archive others
    setVersions((prev) =>
      prev.map((v) => ({
        ...v,
        attributes: {
          ...v.attributes,
          status:
            v.id === versionId
              ? 'published'
              : v.attributes.status === 'published'
              ? 'archived'
              : v.attributes.status,
        },
      }))
    );
    return response.data;
  };

  return {
    schema,
    versions,
    loading,
    publishedVersion: versions.find((v) => v.attributes.status === 'published'),
    draftVersions: versions.filter((v) => v.attributes.status === 'draft'),
    createVersion,
    publishVersion,
  };
}
```

---

## Client Methods

### Form Schemas

```typescript
const client = new FormsClient({ /* config */ });

// List schemas
const { data: schemas } = await client.formSchemas.list();

// Create schema
const { data: schema } = await client.formSchemas.create({
  form_schema: {
    name: 'GAD-7 Anxiety Assessment',
    description: 'Generalized Anxiety Disorder 7-item scale',
  },
});

// Get schema with versions
const { data: schema } = await client.formSchemas.get('schema-uuid', {
  include: 'versions',
});
```

### Schema Versions

```typescript
// Create draft version
const { data: version } = await client.formSchemaVersions.create('schema-uuid', {
  form_schema_version: {
    form_fields: {
      fields: [
        {
          id: 'intro',
          type: 'display',
          label: 'Over the last 2 weeks, how often have you been bothered by:',
        },
        {
          id: 'q1',
          type: 'radio',
          label: 'Feeling nervous, anxious, or on edge',
          options: [
            { value: '0', label: 'Not at all' },
            { value: '1', label: 'Several days' },
            { value: '2', label: 'More than half the days' },
            { value: '3', label: 'Nearly every day' },
          ],
          scores: [0, 1, 2, 3],
        },
        // ... more questions
      ],
      validationRules: {
        required_fields: ['q1', 'q2', 'q3', 'q4', 'q5', 'q6', 'q7'],
      },
      scoringRules: {
        sum: [{ name: 'total', fields: ['q1', 'q2', 'q3', 'q4', 'q5', 'q6', 'q7'], output: 'total_score' }],
        range: [{
          name: 'severity',
          input: 'total_score',
          output: 'severity_level',
          ranges: [
            { min: 0, max: 4, label: 'Minimal anxiety' },
            { min: 5, max: 9, label: 'Mild anxiety' },
            { min: 10, max: 14, label: 'Moderate anxiety' },
            { min: 15, max: 21, label: 'Severe anxiety' },
          ],
        }],
      },
    },
  },
});

// Publish version (archives previous published)
await client.formSchemaVersions.publish('schema-uuid', 'version-uuid');
```

### App Form Instances

```typescript
// Assign form to user
const { data: instance } = await client.appForms.create('form-uuid', {
  app_form_instance: {
    assigned_to_email: 'patient@example.com',
    assigned_to_name: 'John Patient',
    assigned_by_email: 'doctor@clinic.com',
    assigned_by_name: 'Dr. Smith',
    expires_at: '2025-02-01T23:59:59Z',
    metadata: {
      appointment_id: 'apt-123',
      department: 'Mental Health',
    },
  },
});

// Start form (pending → in_progress)
await client.appForms.start('form-uuid', 'instance-uuid');

// Save draft responses
await client.appForms.update('form-uuid', 'instance-uuid', {
  app_form_instance: {
    responses: [null, 2, 1, 3, 2, 1, 2, 1],
  },
});

// Submit form (validates, scores, completes)
const { data: completed } = await client.appForms.submit('form-uuid', 'instance-uuid', {
  responses: [null, 2, 1, 3, 2, 1, 2, 1],
});

console.log(completed.attributes.calculated_scores);
// { total_score: 12, severity_level: 'Moderate anxiety' }
```

---

## GAD-7 Complete Example

```typescript
import React, { useEffect } from 'react';
import { FormsClient } from '@23blocks/forms-sdk';

function GAD7Assessment({ formId, instanceId }: { formId: string; instanceId: string }) {
  const client = new FormsClient({ /* config */ });
  const {
    instance,
    schema,
    loading,
    error,
    start,
    saveDraft,
    submit,
  } = useAppFormInstance({ client, formId, instanceId });

  useEffect(() => {
    // Auto-start if pending
    if (instance?.attributes.status === 'pending') {
      start();
    }
  }, [instance]);

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!instance || !schema) return null;

  const { status, calculated_scores, completed_at } = instance.attributes;

  // Show results if completed
  if (status === 'completed') {
    return (
      <div className="p-6 bg-white rounded-lg shadow">
        <h2 className="text-xl font-bold mb-4">GAD-7 Results</h2>
        <div className="space-y-2">
          <p><strong>Total Score:</strong> {calculated_scores.total_score}/21</p>
          <p><strong>Severity:</strong> {calculated_scores.severity_level}</p>
          <p><strong>Completed:</strong> {formatDate(completed_at)}</p>
        </div>
      </div>
    );
  }

  return (
    <div className="p-6 bg-white rounded-lg shadow">
      <h2 className="text-xl font-bold mb-4">GAD-7 Anxiety Assessment</h2>
      <AppFormRenderer
        schema={schema}
        instance={instance}
        onSave={saveDraft}
        onSubmit={submit}
      />
    </div>
  );
}
```

---

## Scoring Display Component

```typescript
interface ScoreDisplayProps {
  scores: Record<string, unknown>;
  scoringRules?: ScoringRules;
}

function ScoreDisplay({ scores, scoringRules }: ScoreDisplayProps) {
  return (
    <div className="bg-gray-50 p-4 rounded-lg">
      <h3 className="font-semibold mb-3">Assessment Results</h3>
      <dl className="space-y-2">
        {Object.entries(scores).map(([key, value]) => {
          const label = formatScoreLabel(key);
          const severity = getSeverityColor(key, value, scoringRules);

          return (
            <div key={key} className="flex justify-between">
              <dt className="text-gray-600">{label}:</dt>
              <dd className={`font-medium ${severity}`}>
                {String(value)}
              </dd>
            </div>
          );
        })}
      </dl>
    </div>
  );
}

function formatScoreLabel(key: string): string {
  return key
    .replace(/_/g, ' ')
    .replace(/\b\w/g, (c) => c.toUpperCase());
}

function getSeverityColor(
  key: string,
  value: unknown,
  rules?: ScoringRules
): string {
  if (key.includes('severity') || key.includes('level')) {
    const str = String(value).toLowerCase();
    if (str.includes('minimal') || str.includes('low')) return 'text-green-600';
    if (str.includes('mild')) return 'text-yellow-600';
    if (str.includes('moderate')) return 'text-orange-600';
    if (str.includes('severe') || str.includes('high')) return 'text-red-600';
  }
  return 'text-gray-900';
}
```

---

## Metadata Search Utility

```typescript
// Search instances by metadata
async function searchByMetadata(
  client: FormsClient,
  formId: string,
  metadata: Record<string, unknown>
) {
  // Build query params
  const params = new URLSearchParams();

  Object.entries(metadata).forEach(([key, value]) => {
    if (typeof value === 'object' && value !== null) {
      // Complex operator: { op: 'like', value: 'pending' }
      const { op, value: val } = value as { op: string; value: unknown };
      params.append(`metadata_${key}[op]`, op);
      params.append(`metadata_${key}[value]`, String(val));
    } else {
      // Simple equality
      params.append(`metadata_${key}`, String(value));
    }
  });

  return client.appForms.list(formId, { params });
}

// Usage
const instances = await searchByMetadata(client, formId, {
  appointment_id: '7688342',
  department: 'Mental Health',
});

// With LIKE operator
const pending = await searchByMetadata(client, formId, {
  status: { op: 'like', value: 'pend' },
});
```
