---
name: wallet-wallets-api
description: Manage 23blocks digital wallets via REST API. Use when creating wallets, checking balances, processing credit/debit transactions, transferring funds, generating OTP codes, or storing content in wallets.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Wallets API

Complete API reference for 23blocks digital wallet management with transactions, transfers, OTP authorization, and content storage.

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
curl -X GET "$BLOCKS_API_URL/users/{unique_id}/wallets" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /wallets/validate/ - Validate Wallet

Validates a wallet's status and configuration.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/wallets/validate/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "wallet": {
      "wallet_code": "wal-abc-123"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `wallet_code` | string | Yes | Wallet code to validate |

**Response 200:**
```json
{
  "data": {
    "id": "wal-abc-123",
    "type": "wallet",
    "attributes": {
      "wallet_code": "wal-abc-123",
      "status": "active",
      "valid": true,
      "currency": "USD",
      "balance": 250.00
    }
  }
}
```

**Errors:**
- `404 Not Found` - Wallet not found
- `422 Unprocessable Entity` - Invalid wallet code

---

### GET /users/:unique_id/wallets - List User Wallets

Lists all wallets for a specific user with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/wallets?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `unique_id` | string | Yes | User unique identifier |

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
      "id": "wal-abc-123",
      "type": "wallet",
      "attributes": {
        "unique_id": "wal-uuid-123",
        "wallet_code": "wal-abc-123",
        "user_unique_id": "user-uuid-123",
        "balance": 250.00,
        "currency": "USD",
        "status": "active",
        "created_at": "2025-06-15T10:30:00Z",
        "updated_at": "2025-06-15T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 3
  }
}
```

---

### GET /users/:unique_id/wallets/:wallet_code - Get Wallet Details

Retrieves a single wallet by wallet code.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/wallets/wal-abc-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `unique_id` | string | Yes | User unique identifier |
| `wallet_code` | string | Yes | Wallet code |

**Response 200:**
```json
{
  "data": {
    "id": "wal-abc-123",
    "type": "wallet",
    "attributes": {
      "unique_id": "wal-uuid-123",
      "wallet_code": "wal-abc-123",
      "user_unique_id": "user-uuid-123",
      "balance": 250.00,
      "currency": "USD",
      "status": "active",
      "created_at": "2025-06-15T10:30:00Z",
      "updated_at": "2025-06-20T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Wallet not found

---

### POST /users/:unique_id/wallets/ - Create Wallet

Creates a new wallet for a user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/wallets/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "wallet": {
      "currency": "USD",
      "status": "active"
    }
  }'
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `unique_id` | string | Yes | User unique identifier |

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `currency` | string | Yes | Wallet currency code (e.g., USD, EUR) |
| `status` | enum | No | active, suspended, closed (default: active) |

**Response 201:**
```json
{
  "data": {
    "id": "wal-def-456",
    "type": "wallet",
    "attributes": {
      "unique_id": "wal-uuid-456",
      "wallet_code": "wal-def-456",
      "user_unique_id": "user-uuid-123",
      "balance": 0.00,
      "currency": "USD",
      "status": "active",
      "created_at": "2025-06-20T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (invalid currency, duplicate wallet)

---

### PUT /users/:unique_id/wallets/:wallet_code - Update Wallet

Updates an existing wallet.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/wallets/wal-abc-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "wallet": {
      "status": "suspended"
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
| `status` | enum | No | active, suspended, closed |

**Response 200:**
```json
{
  "data": {
    "id": "wal-abc-123",
    "type": "wallet",
    "attributes": {
      "unique_id": "wal-uuid-123",
      "wallet_code": "wal-abc-123",
      "user_unique_id": "user-uuid-123",
      "balance": 250.00,
      "currency": "USD",
      "status": "suspended",
      "updated_at": "2025-06-21T09:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Wallet not found
- `422 Unprocessable Entity` - Invalid status transition

---

### POST /users/:unique_id/wallets/:wallet_code - Create Transaction

Creates a credit or debit transaction on a wallet.

**Request:**
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
| `currency` | string | Yes | Currency code (e.g., USD) |
| `description` | string | No | Transaction description |
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
- `422 Unprocessable Entity` - Insufficient funds (for debit), invalid amount, wallet suspended

---

### GET /users/:unique_id/wallets/:wallet_code/transactions - List Wallet Transactions

Lists all transactions for a wallet with pagination.

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
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 42
  }
}
```

---

### POST /users/:unique_id/wallets/:wallet_code/transfer - Transfer Between Wallets

Transfers funds from one wallet to another.

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

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `unique_id` | string | Yes | Source user unique identifier |
| `wallet_code` | string | Yes | Source wallet code |

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `target_wallet_code` | string | Yes | Target wallet code |
| `amount` | decimal | Yes | Transfer amount (positive value) |
| `currency` | string | Yes | Currency code |
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
- `422 Unprocessable Entity` - Insufficient funds, invalid OTP, wallet suspended, currency mismatch

---

### GET /users/:unique_id/wallets/:wallet_code/content - List Wallet Content

Lists content stored in a wallet.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/wallets/wal-abc-123/content?page=1&records=20" \
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
      "id": "content-uuid-001",
      "type": "wallet_content",
      "attributes": {
        "unique_id": "content-uuid-001",
        "key": "loyalty_tier",
        "value": "gold",
        "metadata": {
          "points": 15000,
          "expires_at": "2026-12-31T23:59:59Z"
        },
        "created_at": "2025-06-15T10:30:00Z",
        "updated_at": "2025-06-20T14:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 2
  }
}
```

---

### PUT /users/:unique_id/wallets/:wallet_code/content - Store Content in Wallet

Stores or updates content associated with a wallet.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/wallets/wal-abc-123/content" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": {
      "key": "loyalty_tier",
      "value": "gold",
      "metadata": {
        "points": 15000,
        "expires_at": "2026-12-31T23:59:59Z"
      }
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
| `key` | string | Yes | Content key identifier |
| `value` | string | Yes | Content value |
| `metadata` | object | No | Additional metadata as JSON object |

**Response 200:**
```json
{
  "data": {
    "id": "content-uuid-001",
    "type": "wallet_content",
    "attributes": {
      "unique_id": "content-uuid-001",
      "key": "loyalty_tier",
      "value": "gold",
      "metadata": {
        "points": 15000,
        "expires_at": "2026-12-31T23:59:59Z"
      },
      "updated_at": "2025-06-20T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Wallet not found
- `422 Unprocessable Entity` - Validation errors

---

### POST /users/:unique_id/wallets/:wallet_code/otp - Generate OTP Authorization Code

Generates a one-time password code for authorizing sensitive wallet operations.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/wallets/wal-abc-123/otp" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "otp": {
      "operation": "transfer",
      "amount": 500.00
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
| `operation` | string | Yes | Operation type requiring OTP (e.g., transfer, debit) |
| `amount` | decimal | No | Amount associated with the operation |

**Response 201:**
```json
{
  "data": {
    "id": "otp-uuid-001",
    "type": "otp",
    "attributes": {
      "unique_id": "otp-uuid-001",
      "operation": "transfer",
      "expires_at": "2025-06-20T12:15:00Z",
      "created_at": "2025-06-20T12:00:00Z"
    }
  },
  "meta": {
    "message": "OTP code sent to registered contact method"
  }
}
```

**Errors:**
- `404 Not Found` - Wallet not found
- `422 Unprocessable Entity` - Invalid operation type
- `429 Too Many Requests` - OTP generation rate limit exceeded

---

### POST /companies/:url_id/wallets/:wallet_code/transactions - Transaction Webhook

Webhook endpoint for external systems to post transaction notifications.

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
      "description": "External payment received",
      "reference_id": "ext-pay-12345"
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
| `amount` | decimal | Yes | Transaction amount |
| `currency` | string | Yes | Currency code |
| `description` | string | No | Transaction description |
| `reference_id` | string | No | External reference ID |

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
      "description": "External payment received",
      "reference_id": "ext-pay-12345",
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
- `422 Unprocessable Entity` - Validation errors

---

## Data Models

### Wallet
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `wallet_code` | string | Human-readable wallet code |
| `user_unique_id` | uuid | Owner user ID |
| `balance` | decimal | Current wallet balance |
| `currency` | string | Currency code (e.g., USD, EUR) |
| `status` | enum | active, suspended, closed |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Transaction
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `transaction_type` | enum | credit, debit, transfer |
| `amount` | decimal | Transaction amount |
| `currency` | string | Currency code |
| `description` | string | Transaction description |
| `reference_id` | string | External reference ID |
| `source_wallet` | string | Source wallet code (for transfers) |
| `target_wallet` | string | Target wallet code |
| `status` | enum | completed, pending, failed |
| `created_at` | timestamp | Creation time |

### WalletContent
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `key` | string | Content key |
| `value` | string | Content value |
| `metadata` | object | Additional metadata |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### OTP
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `operation` | string | Operation type |
| `expires_at` | timestamp | Expiration time |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "insufficient_funds",
    "title": "Insufficient Funds",
    "detail": "The wallet does not have sufficient funds to complete this transaction."
  }]
}
```
