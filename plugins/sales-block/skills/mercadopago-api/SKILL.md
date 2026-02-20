---
name: sales-mercadopago-api
description: Manage 23blocks MercadoPago integration via REST API. Use when configuring MercadoPago settings, creating payments, processing PSE bank transfers, or handling MercadoPago webhooks.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# MercadoPago API

Complete API reference for 23blocks MercadoPago payment gateway integration for Latin American payment methods.

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
curl -X GET "$BLOCKS_API_URL/mercadopago" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /mercadopago - Get Configuration

Retrieves MercadoPago configuration for the current account.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mercadopago" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "mp-config-uuid-001",
    "type": "mercadopago_config",
    "attributes": {
      "unique_id": "mp-config-uuid-001",
      "public_key": "APP_USR-xxxxx-xxxxx",
      "country": "CO",
      "currency": "COP",
      "sandbox_mode": false,
      "webhook_url": "https://sales.api.us.23blocks.com/mercadopago/wh_mp001/webhook",
      "payment_methods_enabled": ["credit_card", "debit_card", "pse", "cash"],
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### POST /mercadopago/payments - Create Payment

Creates a MercadoPago payment.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mercadopago/payments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payment": {
      "transaction_amount": 150000,
      "description": "Premium Plan Subscription",
      "payment_method_id": "visa",
      "token": "card_token_abc123",
      "installments": 1,
      "payer": {
        "email": "user@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "identification": {
          "type": "CC",
          "number": "1234567890"
        }
      },
      "metadata": {
        "order_id": "order-uuid-123"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `transaction_amount` | decimal | Yes | Payment amount |
| `description` | string | Yes | Payment description |
| `payment_method_id` | string | Yes | Payment method (visa, mastercard, pse, etc.) |
| `token` | string | Yes | Card token from MercadoPago.js |
| `installments` | integer | No | Number of installments |
| `payer.email` | string | Yes | Payer email |
| `payer.first_name` | string | No | Payer first name |
| `payer.last_name` | string | No | Payer last name |
| `payer.identification.type` | string | No | ID type (CC, CE, NIT) |
| `payer.identification.number` | string | No | ID number |
| `metadata` | object | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "mp-payment-uuid-001",
    "type": "mercadopago_payment",
    "attributes": {
      "unique_id": "mp-payment-uuid-001",
      "mp_payment_id": 12345678,
      "transaction_amount": 150000,
      "currency": "COP",
      "status": "approved",
      "status_detail": "accredited",
      "payment_method_id": "visa",
      "installments": 1,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Payment rejected or validation error

---

### POST /mercadopago/payments/pse - Create PSE Payment

Creates a PSE (bank transfer) payment specific to Colombia.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mercadopago/payments/pse" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payment": {
      "transaction_amount": 250000,
      "description": "Enterprise Plan Payment",
      "payer": {
        "email": "empresa@example.com",
        "entity_type": "individual",
        "first_name": "Maria",
        "last_name": "Garcia",
        "identification": {
          "type": "CC",
          "number": "9876543210"
        }
      },
      "transaction_details": {
        "financial_institution": "1007"
      },
      "callback_url": "https://app.example.com/payment/callback",
      "metadata": {
        "order_id": "order-uuid-456"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `transaction_amount` | decimal | Yes | Payment amount |
| `description` | string | Yes | Payment description |
| `payer.email` | string | Yes | Payer email |
| `payer.entity_type` | enum | Yes | individual, association |
| `payer.first_name` | string | Yes | Payer first name |
| `payer.last_name` | string | Yes | Payer last name |
| `payer.identification.type` | string | Yes | ID type (CC, CE, NIT) |
| `payer.identification.number` | string | Yes | ID number |
| `transaction_details.financial_institution` | string | Yes | Bank code |
| `callback_url` | string | Yes | Redirect URL after bank authorization |
| `metadata` | object | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "mp-pse-payment-uuid-001",
    "type": "mercadopago_payment",
    "attributes": {
      "unique_id": "mp-pse-payment-uuid-001",
      "mp_payment_id": 87654321,
      "transaction_amount": 250000,
      "currency": "COP",
      "status": "pending",
      "status_detail": "pending_waiting_transfer",
      "payment_method_id": "pse",
      "external_resource_url": "https://www.mercadopago.com.co/sandbox/payments/redirect/...",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /mercadopago/:url_id/webhook - MercadoPago Webhook

Receives MercadoPago payment notifications. This endpoint is called by MercadoPago, not by your application directly.

**Request (from MercadoPago):**
```bash
curl -X POST "$BLOCKS_API_URL/mercadopago/wh_mp001/webhook" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 12345678,
    "type": "payment",
    "action": "payment.updated",
    "data": {
      "id": "12345678"
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

## Data Models

### MercadoPagoConfig
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Internal unique identifier |
| `public_key` | string | MercadoPago public key |
| `country` | string | Country code (CO, AR, MX, BR, etc.) |
| `currency` | string | Default currency |
| `sandbox_mode` | boolean | Whether in sandbox/test mode |
| `webhook_url` | string | Configured webhook URL |
| `payment_methods_enabled` | array | Enabled payment methods |
| `created_at` | timestamp | Creation time |

### MercadoPagoPayment
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Internal unique identifier |
| `mp_payment_id` | integer | MercadoPago payment ID |
| `transaction_amount` | decimal | Payment amount |
| `currency` | string | Currency code |
| `status` | enum | approved, pending, rejected, in_process |
| `status_detail` | string | Detailed status (accredited, pending_waiting_transfer, cc_rejected_other_reason, etc.) |
| `payment_method_id` | string | Payment method used |
| `installments` | integer | Number of installments |
| `external_resource_url` | string | External redirect URL (for PSE) |
| `metadata` | object | Custom metadata |
| `created_at` | timestamp | Creation time |

---

## Common Financial Institutions (PSE - Colombia)

| Code | Bank |
|------|------|
| `1007` | Bancolombia |
| `1009` | Citibank |
| `1012` | Banco GNB Sudameris |
| `1013` | BBVA |
| `1014` | Itau |
| `1023` | Banco de Occidente |
| `1040` | Banco Agrario |
| `1051` | Davivienda |
| `1052` | AV Villas |
| `1062` | Banco Falabella |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "payment_rejected",
    "title": "Payment Rejected",
    "detail": "The payment was rejected by the card issuer. Status: cc_rejected_other_reason."
  }]
}
```
