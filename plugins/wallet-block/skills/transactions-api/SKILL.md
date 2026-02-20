---
name: wallet-transactions-api
description: Manage 23blocks wallet transactions via REST API. Use when creating credit/debit transactions, listing transaction history, transferring funds between wallets, or integrating with transaction webhooks.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Transactions API

Complete API reference for 23blocks wallet transaction processing including credits, debits, transfers, transaction history, and webhook integration.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://wallet.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Wallet API base URL | `https://wallet.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/users/{unique_id}/wallets/{wallet_code}/transactions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /users/:unique_id/wallets/:wallet_code - Create Transaction

Creates a credit or debit transaction on a wallet. Credit transactions add funds; debit transactions remove funds.

**Request (Credit):**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/wallets/wal-abc-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transaction": {
      "transaction_type": "credit",
      "amount": 100.00,
      "currency": "USD",
      "description": "Deposit from bank transfer",
      "reference_id": "ext-ref-001"
    }
  }'
```

**Request (Debit):**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/wallets/wal-abc-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transaction": {
      "transaction_type": "debit",
      "amount": 25.50,
      "currency": "USD",
      "description": "Purchase - Order #12345",
      "reference_id": "order-12345"
    }
  }'
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `unique_id` | string | Yes | User unique identifier |
| `wallet_code` | string | Yes | Wallet code |

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `transaction_type` | enum | Yes | credit, debit |
| `amount` | decimal | Yes | Transaction amount (positive value) |
| `currency` | string | Yes | Currency code (e.g., USD, EUR) |
| `description` | string | No | Human-readable transaction description |
| `reference_id` | string | No | External reference ID for reconciliation |

**Response 201:**
```json
{
  "data": {
    "id": "txn-uuid-789",
    "type": "transaction",
    "attributes": {
      "unique_id": "txn-uuid-789",
      "transaction_type": "credit",
      "amount": 100.00,
      "currency": "USD",
      "description": "Deposit from bank transfer",
      "reference_id": "ext-ref-001",
      "source_wallet": null,
      "target_wallet": "wal-abc-123",
      "status": "completed",
      "created_at": "2025-06-20T11:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Wallet not found
- `422 Unprocessable Entity` - Insufficient funds (debit), invalid amount, wallet suspended/closed

---

### GET /users/:unique_id/wallets/:wallet_code/transactions - List Transactions

Lists all transactions for a wallet with pagination, ordered by most recent first.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/wallets/wal-abc-123/transactions?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `unique_id` | string | Yes | User unique identifier |
| `wallet_code` | string | Yes | Wallet code |

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "txn-uuid-789",
      "type": "transaction",
      "attributes": {
        "unique_id": "txn-uuid-789",
        "transaction_type": "credit",
        "amount": 100.00,
        "currency": "USD",
        "description": "Deposit from bank transfer",
        "reference_id": "ext-ref-001",
        "source_wallet": null,
        "target_wallet": "wal-abc-123",
        "status": "completed",
        "created_at": "2025-06-20T11:00:00Z"
      }
    },
    {
      "id": "txn-uuid-790",
      "type": "transaction",
      "attributes": {
        "unique_id": "txn-uuid-790",
        "transaction_type": "debit",
        "amount": 25.50,
        "currency": "USD",
        "description": "Purchase - Order #12345",
        "reference_id": "order-12345",
        "source_wallet": "wal-abc-123",
        "target_wallet": null,
        "status": "completed",
        "created_at": "2025-06-19T09:30:00Z"
      }
    },
    {
      "id": "txn-uuid-791",
      "type": "transaction",
      "attributes": {
        "unique_id": "txn-uuid-791",
        "transaction_type": "transfer",
        "amount": 50.00,
        "currency": "USD",
        "description": "Payment for services",
        "reference_id": "inv-2025-001",
        "source_wallet": "wal-abc-123",
        "target_wallet": "wal-xyz-789",
        "status": "completed",
        "created_at": "2025-06-18T14:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### POST /users/:unique_id/wallets/:wallet_code/transfer - Transfer Between Wallets

Transfers funds from the source wallet to a target wallet. Creates two linked transaction records (debit on source, credit on target).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/wallets/wal-abc-123/transfer" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transfer": {
      "target_wallet_code": "wal-xyz-789",
      "amount": 50.00,
      "currency": "USD",
      "description": "Payment for services",
      "reference_id": "inv-2025-001"
    }
  }'
```

**Request with OTP (for high-value transfers):**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/wallets/wal-abc-123/transfer" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transfer": {
      "target_wallet_code": "wal-xyz-789",
      "amount": 5000.00,
      "currency": "USD",
      "description": "Large payment - Invoice #9001",
      "reference_id": "inv-9001",
      "otp_code": "482917"
    }
  }'
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `unique_id` | string | Yes | Source user unique identifier |
| `wallet_code` | string | Yes | Source wallet code |

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `target_wallet_code` | string | Yes | Target wallet code to receive funds |
| `amount` | decimal | Yes | Transfer amount (positive value) |
| `currency` | string | Yes | Currency code (must match both wallets) |
| `description` | string | No | Transfer description |
| `reference_id` | string | No | External reference ID |
| `otp_code` | string | No | OTP code for high-value transfers |

**Response 201:**
```json
{
  "data": {
    "id": "txn-uuid-transfer-001",
    "type": "transaction",
    "attributes": {
      "unique_id": "txn-uuid-transfer-001",
      "transaction_type": "transfer",
      "amount": 50.00,
      "currency": "USD",
      "description": "Payment for services",
      "reference_id": "inv-2025-001",
      "source_wallet": "wal-abc-123",
      "target_wallet": "wal-xyz-789",
      "status": "completed",
      "created_at": "2025-06-20T12:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Source or target wallet not found
- `422 Unprocessable Entity` - Insufficient funds, invalid OTP code, wallets suspended/closed, currency mismatch, self-transfer

---

### POST /companies/:url_id/wallets/:wallet_code/transactions - Transaction Webhook

Webhook endpoint for external systems to post transaction notifications to a wallet. Used for integrating external payment processors, banking systems, or third-party services.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/companies/my-company/wallets/wal-abc-123/transactions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transaction": {
      "transaction_type": "credit",
      "amount": 200.00,
      "currency": "USD",
      "description": "External payment received via Stripe",
      "reference_id": "stripe-pi-12345abc"
    }
  }'
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url_id` | string | Yes | Company URL identifier |
| `wallet_code` | string | Yes | Target wallet code |

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `transaction_type` | enum | Yes | credit, debit |
| `amount` | decimal | Yes | Transaction amount (positive value) |
| `currency` | string | Yes | Currency code |
| `description` | string | No | Transaction description |
| `reference_id` | string | No | External reference ID for reconciliation |

**Response 201:**
```json
{
  "data": {
    "id": "txn-uuid-webhook-001",
    "type": "transaction",
    "attributes": {
      "unique_id": "txn-uuid-webhook-001",
      "transaction_type": "credit",
      "amount": 200.00,
      "currency": "USD",
      "description": "External payment received via Stripe",
      "reference_id": "stripe-pi-12345abc",
      "source_wallet": null,
      "target_wallet": "wal-abc-123",
      "status": "completed",
      "created_at": "2025-06-20T13:00:00Z"
    }
  }
}
```

**Errors:**
- `401 Unauthorized` - Invalid webhook credentials
- `404 Not Found` - Company or wallet not found
- `422 Unprocessable Entity` - Validation errors, duplicate reference_id

---

## Data Models

### Transaction
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `transaction_type` | enum | credit, debit, transfer |
| `amount` | decimal | Transaction amount (always positive) |
| `currency` | string | Currency code (e.g., USD, EUR) |
| `description` | string | Human-readable description |
| `reference_id` | string | External reference for reconciliation |
| `source_wallet` | string | Source wallet code (null for credits) |
| `target_wallet` | string | Target wallet code (null for debits) |
| `status` | enum | completed, pending, failed |
| `created_at` | timestamp | Creation time |

### Transfer (extends Transaction)
| Field | Type | Description |
|-------|------|-------------|
| `transaction_type` | enum | Always "transfer" |
| `source_wallet` | string | Wallet debited |
| `target_wallet` | string | Wallet credited |
| `otp_code` | string | OTP code used (if applicable) |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "insufficient_funds",
    "title": "Insufficient Funds",
    "detail": "The wallet does not have sufficient funds to complete this transaction. Current balance: 45.00 USD, requested: 100.00 USD."
  }]
}
```

### Common Transaction Errors

| Code | Status | Description |
|------|--------|-------------|
| `insufficient_funds` | 422 | Wallet balance is less than debit/transfer amount |
| `wallet_suspended` | 422 | Wallet is suspended and cannot process transactions |
| `wallet_closed` | 422 | Wallet is closed and cannot process transactions |
| `invalid_currency` | 422 | Transaction currency does not match wallet currency |
| `invalid_otp` | 422 | OTP code is invalid or expired |
| `self_transfer` | 422 | Cannot transfer to the same wallet |
| `duplicate_reference` | 422 | Reference ID already used for another transaction |
| `wallet_not_found` | 404 | Specified wallet does not exist |
