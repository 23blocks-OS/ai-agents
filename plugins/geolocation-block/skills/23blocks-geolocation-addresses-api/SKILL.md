---
name: 23blocks-geolocation-addresses-api
description: "Geolocation Block addresses: CRUD, user and contact addresses, default, tags. Use for postal addresses."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Addresses API

Complete API reference for 23blocks address management with user/contact associations, default address support, and tags.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://geolocation.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/addresses/:unique_id/` | Get a single address |
| GET | `/addresses/owned_by/:unique_id` | Get all addresses by owner |
| POST | `/addresses/` | Create an address |
| PUT | `/addresses/:unique_id` | Update an address |
| DELETE | `/addresses/:unique_id` | Delete an address |
| POST | `/addresses/:unique_id/tags/` | Add tag to address |
| DELETE | `/addresses/:unique_id/tags/:tag_unique_id` | Remove tag from address |
| GET | `/users/:unique_id/addresses` | Get all addresses for a user |
| GET | `/users/:unique_id/default` | Get user's default address |
| GET | `/contacts/:unique_id/addresses` | Get all addresses for a contact |

---

## Data Models

### Address
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `street` | string | Street address |
| `city` | string | City name |
| `state` | string | State/province code |
| `zip_code` | string | Postal/zip code |
| `country` | string | Country code (ISO 3166) |
| `latitude` | float | Latitude coordinate |
| `longitude` | float | Longitude coordinate |
| `address_type` | string | Type (home, work, billing, shipping) |
| `is_default` | boolean | Whether this is the default address |
| `owner_type` | string | Owner entity type (user, contact) |
| `owner_unique_id` | uuid | Owner entity unique ID |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Address Not Found","detail":"The requested address could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-geolocation`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useGeolocationBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// AddressesService — client.geolocation.addresses
client.geolocation.addresses.get(uniqueId: string): Promise<Address>;
client.geolocation.addresses.getByOwner(ownerUniqueId: string): Promise<Address[]>;
client.geolocation.addresses.create(data: CreateAddressRequest): Promise<Address>;
client.geolocation.addresses.update(uniqueId: string, data: UpdateAddressRequest): Promise<Address>;
client.geolocation.addresses.delete(uniqueId: string): Promise<void>;
client.geolocation.addresses.setDefault(uniqueId: string): Promise<Address>;
```

### TypeScript Types

```typescript
import type {
  Address,
  CreateAddressRequest,
  UpdateAddressRequest,
  ListAddressesParams,
} from '@23blocks/block-geolocation';
```
