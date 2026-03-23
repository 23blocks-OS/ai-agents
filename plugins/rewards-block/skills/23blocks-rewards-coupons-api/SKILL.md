---
name: 23blocks-rewards-coupons-api
description: Manage 23blocks coupons via REST API. Use when creating coupon configurations, generating single or batch coupons, loading coupon codes, previewing and redeeming coupons, voiding coupons or batches.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Coupons API

Complete API reference for 23blocks coupon management with configuration templates, batch generation, and redemption.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Rewards API base URL | `https://rewards.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://rewards.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://rewards.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/configurations/` | List coupon configurations |
| GET | `/configurations/:unique_id` | Get a configuration |
| GET | `/configurations/:unique_id/coupons` | List coupons for a configuration |
| POST | `/configurations/` | Create a configuration |
| PUT | `/configurations/` | Update a configuration |
| DELETE | `/configurations/:unique_id` | Delete a configuration |
| POST | `/configurations/:unique_id/one` | Generate a single coupon |
| POST | `/configurations/:unique_id/batch` | Generate a batch of coupons |
| PUT | `/configurations/:unique_id/batches/:batch_id/void` | Void a coupon batch |
| POST | `/configurations/:unique_id/load` | Load pre-existing coupon codes |
| GET | `/coupons` | List all coupons |
| GET | `/coupons/:unique_id` | Get a coupon |
| POST | `/coupons/` | Create a standalone coupon |
| PUT | `/coupons/:unique_id` | Update a coupon |
| PUT | `/coupons/:unique_id/void` | Void a coupon |
| DELETE | `/coupons/:unique_id` | Delete a coupon |
| POST | `/coupon/preview` | Preview a coupon redemption |
| POST | `/coupon/redeem` | Redeem a coupon |

---

## Data Models

### Configuration
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Configuration name |
| `discount_type` | enum | percentage, fixed |
| `discount_value` | decimal | Discount amount or percentage |
| `max_uses` | integer | Maximum total uses |
| `valid_from` | timestamp | Start of validity |
| `valid_to` | timestamp | End of validity |
| `status` | enum | active, inactive |
| `total_coupons` | integer | Total coupons generated |
| `total_redeemed` | integer | Total coupons redeemed |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Coupon
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `code` | string | Unique coupon code |
| `configuration_id` | uuid | Parent configuration ID |
| `used_count` | integer | Number of times used |
| `max_uses` | integer | Maximum allowed uses |
| `status` | enum | active, redeemed, voided, expired |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### CouponBatch
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `configuration_id` | uuid | Parent configuration ID |
| `quantity` | integer | Number of coupons in batch |
| `status` | enum | completed, voided |
| `voided_count` | integer | Number of voided coupons |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "coupon_expired",
    "title": "Coupon Expired",
    "detail": "The coupon code has expired and can no longer be redeemed."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-rewards
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
// CouponsService — client.rewards.coupons
client.rewards.coupons.list(params?: ListCouponsParams): Promise<PageResult<Coupon>>;
client.rewards.coupons.get(uniqueId: string): Promise<Coupon>;
client.rewards.coupons.getByCode(code: string): Promise<Coupon>;
client.rewards.coupons.create(data: CreateCouponRequest): Promise<Coupon>;
client.rewards.coupons.update(uniqueId: string, data: UpdateCouponRequest): Promise<Coupon>;
client.rewards.coupons.delete(uniqueId: string): Promise<void>;
client.rewards.coupons.validate(data: ValidateCouponRequest): Promise<CouponValidationResult>;
client.rewards.coupons.apply(data: ApplyCouponRequest): Promise<CouponApplication>;
```

### TypeScript Types

```typescript
import type {
  Coupon,
  CouponApplication,
  CreateCouponRequest,
  UpdateCouponRequest,
  ListCouponsParams,
  ValidateCouponRequest,
  CouponValidationResult,
  ApplyCouponRequest,
  DiscountType,
} from '@23blocks/block-rewards';
```

### React Hook

```typescript
import { useRewardsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useRewardsBlock();
  const result = await client.rewards.coupons.list();
}
```
