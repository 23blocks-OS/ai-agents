---
name: 23blocks-wallet-api
description: Manage 23blocks digital wallets via REST API. Use when creating wallets, checking balances, processing credit/debit transactions, transferring funds, generating OTP codes, or storing content in wallets.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Wallets API

Complete API reference for 23blocks digital wallet management with transactions, transfers, OTP authorization, and content storage.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Wallet API base URL | `https://wallet.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://wallet.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://wallet.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/wallets/validate/` | Validate a wallet |
| GET | `/users/:unique_id/wallets` | List user wallets |
| GET | `/users/:unique_id/wallets/:wallet_code` | Get wallet details |
| POST | `/users/:unique_id/wallets/` | Create wallet |
| PUT | `/users/:unique_id/wallets/:wallet_code` | Update wallet |
| POST | `/users/:unique_id/wallets/:wallet_code` | Create transaction (credit/debit) |
| GET | `/users/:unique_id/wallets/:wallet_code/transactions` | List wallet transactions |
| POST | `/users/:unique_id/wallets/:wallet_code/transfer` | Transfer between wallets |
| GET | `/users/:unique_id/wallets/:wallet_code/content` | List wallet content |
| PUT | `/users/:unique_id/wallets/:wallet_code/content` | Store content in wallet |
| POST | `/users/:unique_id/wallets/:wallet_code/otp` | Generate OTP code |
| POST | `/companies/:url_id/wallets/:wallet_code/transactions` | Transaction webhook |

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

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-wallet
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
// WalletsService — client.wallet.wallets
client.wallet.wallets.list(params?: ListWalletsParams): Promise<PageResult<Wallet>>;
client.wallet.wallets.get(uniqueId: string): Promise<Wallet>;
client.wallet.wallets.getByUser(userUniqueId: string): Promise<Wallet>;
client.wallet.wallets.getUserWallet(userUniqueId: string, walletCode: string): Promise<Wallet>;
client.wallet.wallets.create(data: CreateWalletRequest): Promise<Wallet>;
client.wallet.wallets.update(uniqueId: string, data: UpdateWalletRequest): Promise<Wallet>;
client.wallet.wallets.credit(uniqueId: string, data: CreditWalletRequest): Promise<Transaction>;
client.wallet.wallets.debit(uniqueId: string, data: DebitWalletRequest): Promise<Transaction>;
client.wallet.wallets.getBalance(uniqueId: string): Promise<{ balance: number; currency: string }>;
client.wallet.wallets.validate(data: ValidateWalletRequest): Promise<ValidateWalletResponse>;
client.wallet.wallets.transfer(userUniqueId: string, walletCode: string, data: TransferWalletRequest): Promise<Transaction>;
client.wallet.wallets.getContent(userUniqueId: string, walletCode: string): Promise<WalletContent[]>;
client.wallet.wallets.storeContent(userUniqueId: string, walletCode: string, data: StoreWalletContentRequest): Promise<WalletContent>;
client.wallet.wallets.listTransactions(userUniqueId: string, walletCode: string): Promise<PageResult<Transaction>>;
```

### TypeScript Types

```typescript
import type {
  Wallet,
  CreateWalletRequest,
  UpdateWalletRequest,
  ListWalletsParams,
  CreditWalletRequest,
  DebitWalletRequest,
  TransferWalletRequest,
  ValidateWalletRequest,
  ValidateWalletResponse,
  WalletContent,
  StoreWalletContentRequest,
} from '@23blocks/block-wallet';
```

### React Hook

```typescript
import { useWalletBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useWalletBlock();
  const result = await client.wallet.wallets.list();
}
```
