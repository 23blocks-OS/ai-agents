---
name: 23blocks-onboarding-api
description: "Onboarding Block onboarding definitions and steps, plus admin step progression. Use when designing an onboarding."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Onboardings API

Complete API reference for 23blocks onboarding definition management with step configuration and admin user progression.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://onboarding.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

Request and response detail for each endpoint: [ENDPOINTS.md](ENDPOINTS.md).

| Method | Path | Description |
|--------|------|-------------|
| GET | `/onboardings` | List Onboardings |
| GET | `/onboardings/:unique_id/` | Get Onboarding with Steps |
| POST | `/onboardings/` | Create Onboarding |
| PUT | `/onboardings/:unique_id/steps` | Add Step |
| PUT | `/onboardings/:unique_id/steps/:step_unique_id` | Update Step |
| DELETE | `/onboardings/:unique_id/steps/:step_unique_id` | Delete Step |
| PUT | `/onboardings/:unique_id/users/:user_unique_id/` | Admin Progress Step for User |

## Data Models

### Onboarding
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Onboarding name |
| `description` | string | Onboarding description |
| `steps` | array | Ordered array of Step objects |
| `status` | enum | active, inactive |
| `steps_count` | integer | Number of steps |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Step
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Step name |
| `description` | string | Step description |
| `order` | integer | Position in sequence |
| `step_type` | string | Type: action, verification, information |
| `required` | boolean | Whether step is mandatory |
| `metadata` | object | Custom step configuration |

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"403","code":"forbidden","title":"Insufficient Permissions","detail":"Creating onboardings requires role_id 2."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-onboarding`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// OnboardingsService — client.onboarding.onboardings
client.onboarding.onboardings.list(params?: ListOnboardingsParams): Promise<PageResult<Onboarding>>;
client.onboarding.onboardings.get(uniqueId: string): Promise<Onboarding>;
client.onboarding.onboardings.create(data: CreateOnboardingRequest): Promise<Onboarding>;
client.onboarding.onboardings.update(uniqueId: string, data: UpdateOnboardingRequest): Promise<Onboarding>;
client.onboarding.onboardings.delete(uniqueId: string): Promise<void>;
client.onboarding.onboardings.addStep(uniqueId: string, data: AddStepRequest): Promise<OnboardingStep>;
client.onboarding.onboardings.updateStep(uniqueId: string, stepUniqueId: string, data: UpdateStepRequest): Promise<OnboardingStep>;
client.onboarding.onboardings.deleteStep(uniqueId: string, stepUniqueId: string): Promise<void>;
client.onboarding.onboardings.stepUser(uniqueId: string, userUniqueId: string, stepData?: Record<string, unknown>): Promise<Onboarding>;
```

### TypeScript Types

```typescript
import type {
  Onboarding,
  CreateOnboardingRequest,
  UpdateOnboardingRequest,
  ListOnboardingsParams,
  OnboardingStep,
  AddStepRequest,
  UpdateStepRequest,
} from '@23blocks/block-onboarding';
```

### React Hook

```typescript
import { useOnboardingBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useOnboardingBlock();
  const result = await client.onboarding.onboardings.list({ page: 1, perPage: 20 });
}
```
