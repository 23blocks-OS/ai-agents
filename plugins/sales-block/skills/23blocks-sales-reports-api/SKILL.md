---
name: 23blocks-sales-reports-api
description: "Sales Block reports across orders, payments, subscriptions, vendors, providers. Use for sales analytics."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Reports API

Complete API reference for 23blocks sales reporting including orders, payments, subscriptions, vendors, and providers.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://sales.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/reports/orders/summary` | Order summary report |
| POST | `/reports/orders/list` | Order list report |
| POST | `/reports/orders/providers/list` | Provider list report |
| POST | `/reports/orders/providers/summary` | Provider summary report |
| POST | `/reports/flexible_orders/summary` | Flexible order summary report |
| POST | `/reports/payments/list` | Payment list report |
| POST | `/reports/payments/summary` | Payment summary report |
| POST | `/reports/vendors/payments/list` | Vendor payment list report |
| POST | `/reports/vendors/payments/summary` | Vendor payment summary report |
| POST | `/reports/users/subscriptions/list` | Subscription list report |
| POST | `/reports/users/subscriptions/summary` | Subscription summary report |

All report endpoints accept: `start_date`, `end_date`, `group_by` (day/week/month), `status`, `currency`.

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"invalid_parameters","title":"Invalid Report Parameters","detail":"start_date must be before end_date."}]}`.
