---
name: sales-stripe-api
description: Manage 23blocks Stripe integration via REST API. Use when creating Stripe customers, checkout sessions, payments, subscriptions, webhooks, or generating customer portal links.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Stripe API

Complete API reference for 23blocks Stripe payment gateway integration including customers, checkout sessions, payments, subscriptions, and webhooks.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://sales.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

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

---

## Customers

### POST /stripe/customers - Create Stripe Customer

Creates a new customer in Stripe.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stripe/customers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "customer": {
      "email": "user@example.com",
      "name": "John Doe",
      "metadata": {
        "user_id": "user-uuid-456"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Customer email |
| `name` | string | No | Customer name |
| `phone` | string | No | Phone number |
| `metadata` | object | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "stripe-cust-uuid-001",
    "type": "stripe_customer",
    "attributes": {
      "unique_id": "stripe-cust-uuid-001",
      "stripe_customer_id": "cus_Abc123XYZ",
      "email": "user@example.com",
      "name": "John Doe",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /stripe/customers/:unique_id/portal - Customer Portal

Generates a Stripe customer billing portal URL.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stripe/customers/stripe-cust-uuid-001/portal" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "portal": {
      "return_url": "https://app.example.com/account"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `return_url` | string | Yes | URL to redirect after portal session |

**Response 200:**
```json
{
  "data": {
    "type": "portal_session",
    "attributes": {
      "url": "https://billing.stripe.com/session/sess_xyz123",
      "return_url": "https://app.example.com/account",
      "expires_at": "2025-01-12T11:30:00Z"
    }
  }
}
```

---

## Checkout Sessions

### POST /stripe/sessions - Create Checkout Session

Creates a Stripe checkout session for secure payment collection.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stripe/sessions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "session": {
      "customer_id": "cus_Abc123XYZ",
      "success_url": "https://app.example.com/success",
      "cancel_url": "https://app.example.com/cancel",
      "line_items": [
        {
          "price_id": "price_1234567890",
          "quantity": 1
        }
      ],
      "mode": "payment"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `customer_id` | string | No | Stripe customer ID |
| `success_url` | string | Yes | Redirect URL on success |
| `cancel_url` | string | Yes | Redirect URL on cancel |
| `line_items` | array | Yes | Array of line items |
| `line_items[].price_id` | string | Yes | Stripe price ID |
| `line_items[].quantity` | integer | Yes | Item quantity |
| `mode` | enum | Yes | payment, subscription, setup |
| `metadata` | object | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "session-uuid-001",
    "type": "checkout_session",
    "attributes": {
      "unique_id": "session-uuid-001",
      "stripe_session_id": "cs_test_abc123",
      "url": "https://checkout.stripe.com/pay/cs_test_abc123",
      "status": "open",
      "mode": "payment",
      "expires_at": "2025-01-12T11:30:00Z",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### GET /stripe/sessions/:session_id - Get Session

Retrieves a checkout session.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/stripe/sessions/session-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "session-uuid-001",
    "type": "checkout_session",
    "attributes": {
      "unique_id": "session-uuid-001",
      "stripe_session_id": "cs_test_abc123",
      "status": "complete",
      "payment_status": "paid",
      "amount_total": 9999,
      "currency": "usd",
      "customer_email": "user@example.com",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Payments

### POST /stripe/payments - Create Payment

Creates a Stripe payment.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stripe/payments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payment": {
      "amount": 9999,
      "currency": "usd",
      "customer_id": "cus_Abc123XYZ",
      "payment_method": "pm_card_visa",
      "description": "Order payment"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount` | integer | Yes | Amount in cents |
| `currency` | string | Yes | Currency code |
| `customer_id` | string | No | Stripe customer ID |
| `payment_method` | string | No | Stripe payment method ID |
| `description` | string | No | Payment description |
| `metadata` | object | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "stripe-payment-uuid-001",
    "type": "stripe_payment",
    "attributes": {
      "unique_id": "stripe-payment-uuid-001",
      "stripe_payment_id": "pi_3N1234567890",
      "amount": 9999,
      "currency": "usd",
      "status": "succeeded",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Webhooks

### POST /stripe/:url_id/webhook - Stripe Webhook

Receives Stripe webhook events. This endpoint is called by Stripe, not by your application directly.

**Request (from Stripe):**
```bash
curl -X POST "$BLOCKS_API_URL/stripe/wh_abc123/webhook" \
  -H "Stripe-Signature: t=1234567890,v1=signature_hash" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "evt_1234567890",
    "type": "checkout.session.completed",
    "data": {
      "object": {
        "id": "cs_test_abc123",
        "status": "complete"
      }
    }
  }'
```

**Response 200:**
```json
{
  "received": true
}
```

---

### GET /stripe/webhooks - List Webhooks

Lists configured Stripe webhook endpoints.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/stripe/webhooks" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "webhook-uuid-001",
      "type": "stripe_webhook",
      "attributes": {
        "unique_id": "webhook-uuid-001",
        "url_id": "wh_abc123",
        "url": "https://sales.api.us.23blocks.com/stripe/wh_abc123/webhook",
        "events": ["checkout.session.completed", "payment_intent.succeeded"],
        "status": "enabled",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /stripe/webhooks - Create Webhook

Creates a new Stripe webhook endpoint.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stripe/webhooks" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "webhook": {
      "events": [
        "checkout.session.completed",
        "payment_intent.succeeded",
        "payment_intent.payment_failed",
        "customer.subscription.created",
        "customer.subscription.updated",
        "customer.subscription.deleted"
      ]
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `events` | array | Yes | Stripe event types to listen for |

**Response 201:**
```json
{
  "data": {
    "id": "webhook-uuid-new",
    "type": "stripe_webhook",
    "attributes": {
      "unique_id": "webhook-uuid-new",
      "url_id": "wh_def456",
      "url": "https://sales.api.us.23blocks.com/stripe/wh_def456/webhook",
      "events": ["checkout.session.completed", "payment_intent.succeeded"],
      "signing_secret": "whsec_abc123xyz",
      "status": "enabled",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Subscriptions

### GET /stripe/subscriptions - List Stripe Subscriptions

Lists all Stripe subscriptions.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/stripe/subscriptions?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "stripe-sub-uuid-001",
      "type": "stripe_subscription",
      "attributes": {
        "unique_id": "stripe-sub-uuid-001",
        "stripe_subscription_id": "sub_1234567890",
        "stripe_customer_id": "cus_Abc123XYZ",
        "status": "active",
        "current_period_start": "2025-01-01T00:00:00Z",
        "current_period_end": "2025-02-01T00:00:00Z",
        "plan_amount": 4999,
        "plan_currency": "usd",
        "plan_interval": "month",
        "created_at": "2025-01-01T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 45
  }
}
```

---

### POST /stripe/subscriptions - Create Stripe Subscription

Creates a new Stripe subscription.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stripe/subscriptions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "customer_id": "cus_Abc123XYZ",
      "price_id": "price_1234567890",
      "trial_period_days": 14,
      "metadata": {
        "plan_name": "Pro"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `customer_id` | string | Yes | Stripe customer ID |
| `price_id` | string | Yes | Stripe price ID |
| `trial_period_days` | integer | No | Trial period in days |
| `coupon` | string | No | Coupon code |
| `metadata` | object | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "stripe-sub-uuid-new",
    "type": "stripe_subscription",
    "attributes": {
      "unique_id": "stripe-sub-uuid-new",
      "stripe_subscription_id": "sub_new1234567890",
      "status": "trialing",
      "trial_end": "2025-01-26T10:30:00Z",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /stripe/subscriptions/:stripe_subscription_id - Update Subscription

Updates a Stripe subscription.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/stripe/subscriptions/sub_1234567890" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "price_id": "price_new_plan_id",
      "proration_behavior": "create_prorations"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `price_id` | string | No | New Stripe price ID |
| `proration_behavior` | enum | No | create_prorations, none, always_invoice |
| `cancel_at_period_end` | boolean | No | Cancel at end of period |
| `metadata` | object | No | Custom metadata |

**Response 200:**
```json
{
  "data": {
    "id": "stripe-sub-uuid-001",
    "type": "stripe_subscription",
    "attributes": {
      "stripe_subscription_id": "sub_1234567890",
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /stripe/subscriptions/:stripe_subscription_id - Cancel Subscription

Cancels a Stripe subscription.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/stripe/subscriptions/sub_1234567890" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "stripe-sub-uuid-001",
    "type": "stripe_subscription",
    "attributes": {
      "stripe_subscription_id": "sub_1234567890",
      "status": "canceled",
      "canceled_at": "2025-01-12T15:00:00Z"
    }
  }
}
```

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
