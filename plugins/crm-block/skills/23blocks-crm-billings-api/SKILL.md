---
name: 23blocks-crm-billings-api
description: Manage 23blocks CRM billings via REST API. Use when creating meeting billings, tracking outstanding payments, generating revenue/aging/participant reports, and managing payment splits.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Billings API

Complete API reference for 23blocks CRM billing management with financial reporting and payment splits.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Crm API base URL | `https://crm.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://crm.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://crm.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Billing Not Found",
    "detail": "The requested billing record could not be found."
  }]
}
```

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-crm
```

### Setup

```typescript
import { create23BlocksClient } from '@23blocks/sdk';

const client = create23BlocksClient({
  authToken: process.env.BLOCKS_AUTH_TOKEN!,
  apiKey: process.env.BLOCKS_API_KEY!,
  apiUrl: process.env.BLOCKS_API_URL!,
});
```

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
