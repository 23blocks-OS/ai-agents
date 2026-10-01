---
name: 23blocks-onboarding-journeys-api
description: "Onboarding Block a user's journey: start, progress, log, status, suspend, resume. Use while a user goes through onboarding."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Journeys API

Complete API reference for 23blocks user-facing journey progress through onboarding steps.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://onboarding.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/onboard/:unique_id/start` | Start a new onboarding journey |
| GET | `/onboard/:unique_id` | Get journey status |
| GET | `/onboard/:unique_id/details` | Get journey with full step logs |
| PUT | `/onboard/:unique_id` | Progress to next step |
| PUT | `/onboard/:unique_id/log` | Log a step action without progressing |
| PUT | `/onboard/:unique_id/suspend` | Suspend an active journey |
| PUT | `/onboard/:unique_id/resume` | Resume a suspended journey |

---

## Data Models

### Journey
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `onboarding_id` | uuid | Associated onboarding definition |
| `user_unique_id` | uuid | Associated user |
| `current_step` | integer | Current step number (order) |
| `status` | enum | active, completed, suspended |
| `started_at` | timestamp | Journey start time |
| `completed_at` | timestamp | Journey completion time (null if not completed) |
| `logs` | array | Array of JourneyLog entries |

### JourneyLog
| Field | Type | Description |
|-------|------|-------------|
| `step_unique_id` | uuid | Associated step |
| `step_name` | string | Step name at time of log |
| `step_order` | integer | Step order at time of log |
| `action` | string | Action performed (started, completed, field_completed, etc.) |
| `metadata` | object | Additional log metadata |
| `logged_at` | timestamp | Log entry time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"unprocessable_entity","title":"Journey Not Active","detail":"Cannot progress a journey that is not in active status."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-onboarding`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// OnboardService — client.onboarding.onboard
client.onboarding.onboard.start(uniqueId: string, data?: StartOnboardRequest): Promise<OnboardJourney>;
client.onboarding.onboard.get(uniqueId: string): Promise<OnboardJourney>;
client.onboarding.onboard.getDetails(uniqueId: string): Promise<OnboardJourneyDetails>;
client.onboarding.onboard.step(uniqueId: string, data: StepOnboardRequest): Promise<OnboardJourney>;
client.onboarding.onboard.log(uniqueId: string, data: LogOnboardRequest): Promise<OnboardJourney>;
client.onboarding.onboard.suspend(uniqueId: string): Promise<OnboardJourney>;
client.onboarding.onboard.resume(uniqueId: string): Promise<OnboardJourney>;

// UserJourneysService — client.onboarding.userJourneys
client.onboarding.userJourneys.list(params?: ListUserJourneysParams): Promise<PageResult<UserJourney>>;
client.onboarding.userJourneys.get(uniqueId: string): Promise<UserJourney>;
client.onboarding.userJourneys.start(data: StartJourneyRequest): Promise<UserJourney>;
client.onboarding.userJourneys.completeStep(uniqueId: string, data: CompleteStepRequest): Promise<UserJourney>;
client.onboarding.userJourneys.abandon(uniqueId: string): Promise<UserJourney>;
client.onboarding.userJourneys.getByUser(userUniqueId: string): Promise<UserJourney[]>;
client.onboarding.userJourneys.getProgress(uniqueId: string): Promise<{ progress: number; currentStep?: number; completedSteps?: number[] }>;
client.onboarding.userJourneys.listByUserAndOnboarding(userUniqueId: string, onboardingUniqueId: string): Promise<UserJourney>;
client.onboarding.userJourneys.reportList(params: UserJourneyReportParams): Promise<UserJourneyReportList>;
client.onboarding.userJourneys.reportSummary(params: UserJourneyReportParams): Promise<UserJourneyReportSummary>;
```

### TypeScript Types

```typescript
import type {
  OnboardJourney,
  OnboardJourneyDetails,
  OnboardStepDetail,
  OnboardLogEntry,
  StartOnboardRequest,
  StepOnboardRequest,
  LogOnboardRequest,
  UserJourney,
  StartJourneyRequest,
  CompleteStepRequest,
  ListUserJourneysParams,
  UserJourneyReportParams,
  UserJourneyReportSummary,
  UserJourneyReportList,
} from '@23blocks/block-onboarding';
```

### React Hook

```typescript
import { useOnboardingBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useOnboardingBlock();
  const journey = await client.onboarding.onboard.start('onboarding-uuid');
}
```
