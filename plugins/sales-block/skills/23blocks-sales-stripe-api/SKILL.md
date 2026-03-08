---
name: 23blocks-sales-stripe-api
description: Manage 23blocks Stripe integration via REST API. Use when creating Stripe customers, checkout sessions, payments, subscriptions, webhooks, or generating customer portal links.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Stripe API

Complete API reference for 23blocks Stripe payment gateway integration including customers, checkout sessions, payments, subscriptions, and webhooks.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Sales API base URL | `https://sales.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/stripe/subscriptions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/stripe/customers` | Create Stripe customer |
| POST | `/stripe/customers/:unique_id/portal` | Generate customer billing portal URL |
| POST | `/stripe/sessions` | Create checkout session |
| GET | `/stripe/sessions/:session_id` | Get checkout session |
| POST | `/stripe/payments` | Create Stripe payment |
| POST | `/stripe/:url_id/webhook` | Stripe webhook receiver |
| GET | `/stripe/webhooks` | List webhook endpoints |
| POST | `/stripe/webhooks` | Create webhook endpoint |
| GET | `/stripe/subscriptions` | List Stripe subscriptions |
| POST | `/stripe/subscriptions` | Create Stripe subscription |
| PUT | `/stripe/subscriptions/:stripe_subscription_id` | Update Stripe subscription |
| DELETE | `/stripe/subscriptions/:stripe_subscription_id` | Cancel Stripe subscription |

---

## Data Models

### StripeCustomer
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Internal unique identifier |
| `stripe_customer_id` | string | Stripe customer ID (cus_xxx) |
| `email` | string | Customer email |
| `name` | string | Customer name |
| `created_at` | timestamp | Creation time |

### CheckoutSession
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Internal unique identifier |
| `stripe_session_id` | string | Stripe session ID (cs_xxx) |
| `url` | string | Checkout URL |
| `status` | enum | open, complete, expired |
| `payment_status` | enum | paid, unpaid, no_payment_required |
| `mode` | enum | payment, subscription, setup |
| `amount_total` | integer | Total amount in cents |
| `currency` | string | Currency code |
| `expires_at` | timestamp | Session expiration |
| `created_at` | timestamp | Creation time |

### StripeSubscription
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Internal unique identifier |
| `stripe_subscription_id` | string | Stripe subscription ID (sub_xxx) |
| `stripe_customer_id` | string | Stripe customer ID |
| `status` | enum | trialing, active, past_due, canceled, unpaid |
| `plan_amount` | integer | Plan amount in cents |
| `plan_currency` | string | Plan currency |
| `plan_interval` | string | Billing interval |
| `current_period_start` | timestamp | Current period start |
| `current_period_end` | timestamp | Current period end |
| `trial_end` | timestamp | Trial end date |
| `canceled_at` | timestamp | Cancellation time |
| `created_at` | timestamp | Creation time |

### StripeWebhook
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Internal unique identifier |
| `url_id` | string | Webhook URL identifier |
| `url` | string | Full webhook URL |
| `events` | array | Subscribed event types |
| `signing_secret` | string | Webhook signing secret |
| `status` | enum | enabled, disabled |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "stripe_error",
    "title": "Stripe Error",
    "detail": "No such customer: 'cus_invalid'."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-sales
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
// Stripe — client.sales.stripe
client.sales.stripe.createCustomer(data: CreateStripeCustomerRequest): Promise<CreateStripeCustomerResponse>;
client.sales.stripe.createCheckoutSession(data: CreateStripeCheckoutSessionRequest): Promise<StripeCheckoutSession>;
client.sales.stripe.verifySession(sessionId: string): Promise<StripeCheckoutSession>;
client.sales.stripe.createPaymentIntent(data: CreateStripePaymentIntentRequest): Promise<StripePaymentIntent>;
client.sales.stripe.createCustomerPortal(uniqueId: string, data: CreateStripeCustomerPortalRequest): Promise<StripeCustomerPortalSession>;
client.sales.stripe.listSubscriptions(params?: ListStripeSubscriptionsParams): Promise<PageResult<StripeSubscription>>;
client.sales.stripe.createSubscription(data: CreateStripeSubscriptionRequest): Promise<StripeSubscription>;
client.sales.stripe.updateSubscription(stripeSubscriptionId: string, data: UpdateStripeSubscriptionRequest): Promise<StripeSubscription>;
client.sales.stripe.cancelSubscription(stripeSubscriptionId: string): Promise<void>;
client.sales.stripe.listWebhooks(): Promise<StripeWebhook[]>;
client.sales.stripe.createWebhook(data: CreateStripeWebhookRequest): Promise<StripeWebhook>;
```

### TypeScript Types

```typescript
import type {
  CreateStripeCustomerRequest,
  CreateStripeCustomerResponse,
  StripeCheckoutSession,
  CreateStripeCheckoutSessionRequest,
  StripePaymentIntent,
  CreateStripePaymentIntentRequest,
  StripeSubscription,
  CreateStripeSubscriptionRequest,
  UpdateStripeSubscriptionRequest,
  StripeCustomerPortalSession,
  CreateStripeCustomerPortalRequest,
  StripeWebhook,
  CreateStripeWebhookRequest,
  ListStripeSubscriptionsParams,
} from '@23blocks/block-sales';
```

### React Hook

```typescript
import { useSalesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useSalesBlock();
  const result = await client.sales.stripe.listSubscriptions();
}
```
