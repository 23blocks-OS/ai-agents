---
name: 23blocks-sales-providers-api
description: "Sales Block providers and vendor payouts on order details, with reports. Use for marketplace payouts."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Providers API

Complete API reference for 23blocks provider assignment and vendor payment management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://sales.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/sources/:source_id/providers` | Assign provider to a source |
| POST | `/orders/:unique_id/details/:order_details_unique_id/providers` | Assign provider to order detail |
| PUT | `/orders/:unique_id/details/:order_details_unique_id/providers/:provider_unique_id` | Update provider assignment |
| POST | `/orders/:unique_id/details/:detail_unique_id/vendors/:vendor_unique_id/payments` | Create vendor payment |
| PUT | `/orders/:unique_id/details/:detail_unique_id/vendors/:vendor_unique_id/payments/:payment_unique_id` | Update vendor payment |
| PUT | `/orders/:unique_id/details/:detail_unique_id/vendors/:vendor_unique_id/payments/:payment_unique_id/pay` | Execute vendor payment |
| DELETE | `/orders/:unique_id/details/:detail_unique_id/vendors/:vendor_unique_id/payments/:payment_unique_id` | Delete vendor payment |
| GET | `/payables/:payment_unique_id` | Get payable record |
| POST | `/reports/vendors/payments/list` | Vendor payment list report |
| POST | `/reports/vendors/payments/summary` | Vendor payment summary report |
| POST | `/reports/orders/providers/list` | Provider list report |
| POST | `/reports/orders/providers/summary` | Provider summary report |

---

## Data Models

### Provider
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `source_id` | uuid | Source ID |
| `vendor_id` | uuid | Vendor ID |
| `commission_rate` | decimal | Commission percentage |
| `priority` | integer | Provider priority |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |

### OrderProvider
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `order_id` | uuid | Order ID |
| `detail_id` | uuid | Order detail ID |
| `vendor_id` | uuid | Vendor ID |
| `commission_rate` | decimal | Commission percentage |
| `amount` | decimal | Provider amount |
| `status` | enum | assigned, confirmed, completed |
| `created_at` | timestamp | Creation time |

### VendorPayment
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `order_id` | uuid | Order ID |
| `detail_id` | uuid | Order detail ID |
| `vendor_id` | uuid | Vendor ID |
| `amount` | decimal | Payment amount |
| `currency` | string | Currency code |
| `status` | enum | pending, paid, cancelled |
| `payment_method` | string | Payment method |
| `reference` | string | External payment reference |
| `notes` | string | Payment notes |
| `paid_at` | timestamp | Payment execution time |
| `created_at` | timestamp | Creation time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Provider Not Found","detail":"The requested provider could not be found."}]}`.
