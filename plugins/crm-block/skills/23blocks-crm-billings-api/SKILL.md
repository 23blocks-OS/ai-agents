---
name: 23blocks-crm-billings-api
description: "CRM Block meeting billings: outstanding by payer, payment splits, revenue, aging, participant reports. Use for CRM invoicing."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Billings API

Complete API reference for 23blocks CRM billing management with financial reporting and payment splits.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://crm.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/meetings/:meeting_unique_id/billing` | Get meeting billing |
| POST | `/meetings/:meeting_unique_id/billing` | Create meeting billing |
| GET | `/billings/outstanding_by_payer` | Outstanding amounts by payer |
| GET | `/billings/eap_sessions/:participant_email/:payer_name` | EAP session billing |
| GET | `/billings/reports/revenue` | Revenue report |
| GET | `/billings/reports/aging` | Aging report |
| GET | `/billings/reports/participant/:participant_email` | Participant billing report |
| GET | `/billings/:unique_id` | Get billing record |
| PUT | `/billings/:unique_id` | Update billing record |
| DELETE | `/billings/:unique_id` | Delete billing record |
| GET | `/billings/:unique_id/payment_split` | Get payment split details |

---

## Data Models

### Billing
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `meeting_id` | uuid | Associated meeting ID |
| `amount` | decimal | Billing amount |
| `payer_name` | string | Payer name |
| `payer_email` | string | Payer email address |
| `billing_type` | string | Type of billing (session, consultation, etc.) |
| `status` | enum | pending, paid, overdue |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Billing Not Found","detail":"The requested billing record could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-crm`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// MeetingBillingsService — client.crm.billings
list(meetingUniqueId: string, params?: ListMeetingBillingsParams): Promise<PageResult<MeetingBilling>>;
get(uniqueId: string): Promise<MeetingBilling>;
create(meetingUniqueId: string, data: CreateMeetingBillingRequest): Promise<MeetingBilling>;
update(uniqueId: string, data: UpdateMeetingBillingRequest): Promise<MeetingBilling>;
delete(uniqueId: string): Promise<void>;
getPaymentSplit(uniqueId: string): Promise<PaymentSplit[]>;
getEapSessions(participantEmail: string, payerName: string): Promise<EapSession>;
getOutstandingByPayer(): Promise<OutstandingByPayer[]>;
getRevenueReport(): Promise<BillingRevenueReport>;
getAgingReport(): Promise<BillingAgingReport>;
getParticipantReport(participantEmail: string): Promise<BillingParticipantReport>;
```

### TypeScript Types

```typescript
import type {
  MeetingBilling,
  CreateMeetingBillingRequest,
  UpdateMeetingBillingRequest,
  ListMeetingBillingsParams,
  PaymentSplit,
  EapSession,
  OutstandingByPayer,
  BillingRevenueReport,
  BillingAgingReport,
  BillingParticipantReport,
} from '@23blocks/block-crm';
```

### React Hook

```typescript
import { useCrmBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCrmBlock();

  // Example: Get outstanding billing amounts by payer
  const result = await client.crm.billings.getOutstandingByPayer();
}
```
