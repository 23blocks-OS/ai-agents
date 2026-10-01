---
name: 23blocks-sales-stripe-api
description: "Sales Block Stripe: customers, checkout sessions, payments, subscriptions, portal links, webhooks. Use for Stripe payments."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Stripe API

Complete API reference for 23blocks Stripe payment gateway integration including customers, checkout sessions, payments, subscriptions, and webhooks.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://sales.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"stripe_error","title":"Stripe Error","detail":"No such customer: 'cus_invalid'."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-sales`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
