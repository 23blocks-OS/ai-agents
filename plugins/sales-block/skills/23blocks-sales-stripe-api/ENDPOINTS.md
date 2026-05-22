# Stripe API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Customers

### POST /stripe/customers - Create Stripe Customer

Creates a new customer in Stripe.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stripe/customers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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

**Response 201 (raw Stripe JSON):**
```json
{
  "id": "cus_Abc123XYZ",
  "object": "customer",
  "email": "user@example.com",
  "name": "John Doe",
  "metadata": {
    "user_id": "user-uuid-456"
  },
  "created": 1736674200
}
```

> **Note:** Stripe pass-through endpoints return raw Stripe JSON objects, not JSON:API format.

---

### POST /stripe/customers/:unique_id/portal - Customer Portal

Generates a Stripe customer billing portal URL.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stripe/customers/stripe-cust-uuid-001/portal" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "portal": {
      "customer_id": "cus_Abc123XYZ",
      "return_url": "https://app.example.com/account"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `customer_id` | string | Yes | Stripe customer ID |
| `return_url` | string | Yes | URL to redirect after portal session |

**Response 200 (plain JSON):**
```json
{
  "url": "https://billing.stripe.com/session/sess_xyz123"
}
```

> **Note:** Portal returns plain JSON with just the URL, not JSON:API format.

---

## Checkout Sessions

### POST /stripe/sessions - Create Checkout Session

Creates a Stripe checkout session for secure payment collection.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stripe/sessions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "session": {
      "price": "price_1234567890",
      "success_url": "https://app.example.com/success",
      "cancel_url": "https://app.example.com/cancel",
      "customer_id": "cus_Abc123XYZ",
      "mode": "payment"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `price` | string | Yes | Stripe price ID |
| `success_url` | string | Yes | Redirect URL on success |
| `cancel_url` | string | Yes | Redirect URL on cancel |
| `customer_id` | string | No | Stripe customer ID |
| `mode` | enum | No | payment, subscription, setup (default: payment) |
| `quantity` | integer | No | Quantity (default: 1) |
| `metadata` | object | No | Custom metadata |

**Response 201 (raw Stripe JSON):**
```json
{
  "id": "cs_test_abc123",
  "object": "checkout.session",
  "url": "https://checkout.stripe.com/pay/cs_test_abc123",
  "status": "open",
  "payment_status": "unpaid",
  "mode": "payment",
  "amount_total": 9999,
  "currency": "usd",
  "customer": "cus_Abc123XYZ",
  "expires_at": 1736678400,
  "created": 1736674200
}
```

> **Note:** Stripe pass-through endpoints return raw Stripe JSON objects, not JSON:API format.

---

### GET /stripe/sessions/:session_id - Get Session

Retrieves a checkout session.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/stripe/sessions/cs_test_abc123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200 (raw Stripe JSON):**
```json
{
  "id": "cs_test_abc123",
  "object": "checkout.session",
  "status": "complete",
  "payment_status": "paid",
  "amount_total": 9999,
  "currency": "usd",
  "customer_details": {
    "email": "user@example.com"
  },
  "created": 1736674200
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payment": {
      "customer_id": "cus_Abc123XYZ",
      "order_unique_id": "order-uuid-123",
      "amount": 9999,
      "currency": "usd",
      "payment_method": "pm_card_visa",
      "description": "Order payment"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `customer_id` | string | No | Stripe customer ID |
| `order_unique_id` | string | No | 23blocks order unique ID to associate |
| `amount` | integer | Yes | Amount in cents |
| `currency` | string | Yes | Currency code |
| `payment_method` | string | No | Stripe payment method ID |
| `description` | string | No | Payment description |
| `metadata` | object | No | Custom metadata |

**Response 201 (raw Stripe JSON):**
```json
{
  "id": "pi_3N1234567890",
  "object": "payment_intent",
  "amount": 9999,
  "currency": "usd",
  "status": "succeeded",
  "customer": "cus_Abc123XYZ",
  "payment_method": "pm_card_visa",
  "description": "Order payment",
  "created": 1736674200
}
```

> **Note:** Stripe pass-through endpoints return raw Stripe JSON objects, not JSON:API format.

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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "webhook": {
      "url": "https://app.example.com/webhooks/stripe",
      "enabled_events": [
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
| `url` | string | No | Webhook destination URL |
| `enabled_events` | array | Yes | Stripe event types to listen for |

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
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200 (raw Stripe JSON):**
```json
{
  "object": "list",
  "data": [
    {
      "id": "sub_1234567890",
      "object": "subscription",
      "customer": "cus_Abc123XYZ",
      "status": "active",
      "current_period_start": 1735689600,
      "current_period_end": 1738368000,
      "items": {
        "data": [
          {
            "price": {
              "id": "price_1234567890",
              "unit_amount": 4999,
              "currency": "usd",
              "recurring": { "interval": "month" }
            }
          }
        ]
      },
      "created": 1735711800
    }
  ],
  "has_more": true
}
```

> **Note:** Stripe pass-through endpoints return raw Stripe JSON objects, not JSON:API format.

---

### POST /stripe/subscriptions - Create Stripe Subscription

Creates a new Stripe subscription.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stripe/subscriptions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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

**Response 201 (raw Stripe JSON):**
```json
{
  "id": "sub_new1234567890",
  "object": "subscription",
  "customer": "cus_Abc123XYZ",
  "status": "trialing",
  "trial_end": 1737884200,
  "current_period_start": 1736674200,
  "current_period_end": 1739352600,
  "items": {
    "data": [
      {
        "price": {
          "id": "price_1234567890",
          "unit_amount": 4999,
          "currency": "usd"
        }
      }
    ]
  },
  "created": 1736674200
}
```

> **Note:** Stripe pass-through endpoints return raw Stripe JSON objects, not JSON:API format.

---

### PUT /stripe/subscriptions/:stripe_subscription_id - Update Subscription

Updates a Stripe subscription.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/stripe/subscriptions/sub_1234567890" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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

**Response 200 (raw Stripe JSON):**
```json
{
  "id": "sub_1234567890",
  "object": "subscription",
  "status": "active",
  "items": {
    "data": [
      {
        "price": {
          "id": "price_new_plan_id",
          "unit_amount": 7999,
          "currency": "usd"
        }
      }
    ]
  },
  "created": 1735711800
}
```

---

### DELETE /stripe/subscriptions/:stripe_subscription_id - Cancel Subscription

Cancels a Stripe subscription.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/stripe/subscriptions/sub_1234567890" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200 (raw Stripe JSON):**
```json
{
  "id": "sub_1234567890",
  "object": "subscription",
  "status": "canceled",
  "canceled_at": 1736694000,
  "created": 1735711800
}
```

---
