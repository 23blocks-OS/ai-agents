---
name: 23blocks-forms-app-forms-sdk
description: "Forms Block App Forms in TypeScript/React: SDK methods, validation and scoring types. Use for assessment UIs; REST is 23blocks-forms-app-forms-api."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# App Forms SDK (TypeScript/React)

TypeScript types, React components, and patterns for dynamic business forms with validation and scoring.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://forms.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## SDK Methods

> Full SDK method documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Category | Method | Description |
|----------|--------|-------------|
| Form Schemas | `client.formSchemas.list()` | List all form schemas |
| Form Schemas | `client.formSchemas.create(data)` | Create a new schema |
| Form Schemas | `client.formSchemas.get(id, opts)` | Get schema with optional includes |
| Schema Versions | `client.formSchemaVersions.create(schemaId, data)` | Create draft version |
| Schema Versions | `client.formSchemaVersions.list(schemaId)` | List versions for schema |
| Schema Versions | `client.formSchemaVersions.publish(schemaId, versionId)` | Publish version |
| App Form Instances | `client.appForms.create(formId, data)` | Assign form to user |
| App Form Instances | `client.appForms.start(formId, instanceId)` | Start form (pending → in_progress) |
| App Form Instances | `client.appForms.update(formId, instanceId, data)` | Save draft responses |
| App Form Instances | `client.appForms.submit(formId, instanceId, data)` | Submit form (validates, scores) |
| App Form Instances | `client.appForms.cancel(formId, instanceId)` | Cancel instance |
| App Form Instances | `client.appForms.list(formId, params)` | List instances with metadata search |

| Hook | Parameters | Description |
|------|------------|-------------|
| `useAppFormInstance` | `{ client, formId, instanceId }` | Load instance + schema, expose start/save/submit/cancel |
| `useFormSchema` | `client, schemaId` | Load schema + versions, expose createVersion/publishVersion |

| Component | Props | Description |
|-----------|-------|-------------|
| `AppFormRenderer` | `schema, instance, onSave, onSubmit, readOnly?` | Renders dynamic form with validation |
| `FormFieldComponent` | `field, value, onChange, error?, disabled?` | Renders individual field by type |
| `ScoreDisplay` | `scores, scoringRules?` | Displays calculated scores with severity colors |

---

## Data Models

### AppFormInstance
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `assigned_to_email` | string | Assignee email |
| `assigned_to_name` | string | Assignee display name |
| `assigned_by_email` | string | Assigner email |
| `status` | enum | pending, in_progress, completed, cancelled, expired |
| `responses` | array | Responses indexed by field order |
| `calculated_scores` | object | Computed scores after submission |
| `metadata` | object | Custom metadata for search/tracking |
| `magic_link_token` | string | Token for magic link access |
| `expires_at` | timestamp | Expiration time |
| `started_at` | timestamp | When form was started |
| `completed_at` | timestamp | Completion time |

### FormSchemaVersion
| Field | Type | Description |
|-------|------|-------------|
| `version_number` | integer | Sequential version number |
| `status` | enum | draft, published, archived |
| `form_fields` | object | Fields, validationRules, scoringRules |
| `published_at` | timestamp | Publication time |

### Field Types
`text`, `textarea`, `number`, `email`, `phone`, `date`, `time`, `select`, `radio`, `checkbox`, `display`, `signature`, `file`

### Validation Rules
`required_fields`, `min_length`, `max_length`, `pattern` (regex + message), `range` (min/max), `custom` validators

### Scoring Rules
`sum` (add field scores), `average`, `count` (occurrences), `range` (map score to label), `conditional` (if/then), `custom`

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Failed","detail":"..."}]}`.

## SDK Usage (TypeScript)

```bash
npm install @23blocks/forms-sdk
```

```typescript
import { FormsClient } from '@23blocks/forms-sdk';

const client = new FormsClient({
  baseUrl: process.env.BLOCKS_API_URL,
  accessToken: process.env.BLOCKS_AUTH_TOKEN,
  appId: process.env.BLOCKS_API_KEY,
});
```

See [ENDPOINTS.md](ENDPOINTS.md) for full type definitions, React components, hooks, client method examples, and the complete GAD-7 implementation.
