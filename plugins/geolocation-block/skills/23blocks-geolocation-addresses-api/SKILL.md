---
name: 23blocks-geolocation-addresses-api
description: Manage 23blocks addresses via REST API. Use when creating, updating, or deleting addresses, managing user and contact addresses, setting default addresses, or organizing addresses with tags.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Addresses API

Complete API reference for 23blocks address management with user/contact associations, default address support, and tags.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Geolocation API base URL | `https://geolocation.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://geolocation.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://geolocation.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


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
| `owner_id` | uuid | Owner entity unique ID |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Address Not Found",
    "detail": "The requested address could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-geolocation
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
// AddressesService — client.geolocation.addresses
client.geolocation.addresses.list(params?: ListAddressesParams): Promise<PageResult<Address>>;
client.geolocation.addresses.get(uniqueId: string): Promise<Address>;
client.geolocation.addresses.create(data: CreateAddressRequest): Promise<Address>;
client.geolocation.addresses.update(uniqueId: string, data: UpdateAddressRequest): Promise<Address>;
client.geolocation.addresses.delete(uniqueId: string): Promise<void>;
client.geolocation.addresses.recover(uniqueId: string): Promise<Address>;
client.geolocation.addresses.search(query: string, params?: ListAddressesParams): Promise<PageResult<Address>>;
client.geolocation.addresses.listDeleted(params?: ListAddressesParams): Promise<PageResult<Address>>;
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

### React Hook

```typescript
import { useGeolocationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useGeolocationBlock();
  const result = await client.geolocation.addresses.list();
}
```
