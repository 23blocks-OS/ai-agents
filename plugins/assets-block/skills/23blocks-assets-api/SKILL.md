---
name: 23blocks-assets-api
description: "Assets Block asset records: CRUD, parts, maintenance, lending, ownership transfer, images, OTP. Use for an asset's lifecycle."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Assets API

Complete API reference for 23blocks Assets Block asset management with categories, parts, maintenance, lending, ownership transfer, images, and OTP.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://assets.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/assets/` | List all assets |
| GET | `/assets/:unique_id/` | Get a single asset |
| POST | `/assets/` | Create an asset |
| PUT | `/assets/:unique_id` | Update an asset |
| DELETE | `/assets/:unique_id` | Delete an asset (marks enabled=false, status=deleted) |
| GET | `/assets/trash/show` | View trash (deleted assets) |
| POST | `/assets/:unique_id/categories` | Assign category to asset |
| PUT | `/assets/:unique_id/parts` | Update asset parts |
| DELETE | `/assets/:unique_id/parts` | Remove asset parts |
| PUT | `/assets/:unique_id/maintenance` | Set maintenance status |
| POST | `/assets/:unique_id/lend` | Lend asset to a user |
| PUT | `/assets/:unique_id/transfer` | Transfer asset ownership |
| PUT | `/assets/:unique_id/presign` | Presign image upload URL |
| POST | `/assets/:unique_id/images` | Confirm image upload |
| DELETE | `/assets/:unique_id/images/:unique_file_id` | Delete asset image |
| POST | `/assets/:unique_id/otp` | Generate OTP for asset |

---

## Data Models

### Asset
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Asset name |
| `description` | string | Asset description |
| `serial` | string | Serial number |
| `status` | enum | active, inactive, retired, lost, lent, deleted |
| `condition` | enum | new, good, fair, poor |
| `purchase_date` | date | Purchase date |
| `purchase_price` | decimal | Original purchase price |
| `current_value` | decimal | Current estimated value |
| `warranty_expiry` | date | Warranty expiration date |
| `location` | string | Physical location |
| `owner_unique_id` | uuid | Owner user ID |
| `vendor_unique_id` | uuid | Vendor ID |
| `category_unique_id` | uuid | Category ID |
| `maintenance_status` | enum | none, scheduled, in_progress, completed |
| `maintenance_due_at` | timestamp | Maintenance due date |
| `lent_to_unique_id` | uuid | User the asset is lent to |
| `images_count` | integer | Number of images |
| `payload` | jsonb | Custom metadata |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Part
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Part name |
| `serial` | string | Part serial number |
| `description` | string | Part description |
| `asset_unique_id` | uuid | Parent asset ID |
| `created_at` | timestamp | Creation time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Asset Not Found","detail":"The requested asset could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-assets`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useAssetsBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// AssetsService — client.assets.assets
client.assets.assets.list(params?: ListAssetsParams): Promise<PageResult<Asset>>;
client.assets.assets.get(uniqueId: string): Promise<Asset>;
client.assets.assets.create(data: CreateAssetRequest): Promise<Asset>;
client.assets.assets.update(uniqueId: string, data: UpdateAssetRequest): Promise<Asset>;
client.assets.assets.delete(uniqueId: string): Promise<void>;
client.assets.assets.listTrash(): Promise<PageResult<Asset>>;
client.assets.assets.transfer(uniqueId: string, data: TransferAssetRequest): Promise<Asset>;
client.assets.assets.assign(uniqueId: string, data: AssignAssetRequest): Promise<Asset>;
client.assets.assets.unassign(uniqueId: string): Promise<Asset>;
client.assets.assets.listByLocation(locationUniqueId: string, params?: ListAssetsParams): Promise<PageResult<Asset>>;
client.assets.assets.listByAssignee(assignedToUniqueId: string, params?: ListAssetsParams): Promise<PageResult<Asset>>;
client.assets.assets.addToCategory(uniqueId: string, data: AddToCategoryRequest): Promise<Asset>;
client.assets.assets.addParts(uniqueId: string, data: AddPartsRequest): Promise<Asset>;
client.assets.assets.removeParts(uniqueId: string, data: RemovePartsRequest): Promise<Asset>;
client.assets.assets.updateMaintenance(uniqueId: string, data: UpdateMaintenanceRequest): Promise<Asset>;
client.assets.assets.lend(uniqueId: string, data: LendAssetRequest): Promise<Asset>;
client.assets.assets.createOTP(uniqueId: string, data?: CreateOTPRequest): Promise<OTPResponse>;
```

### TypeScript Types

```typescript
import type {
  Asset,
  CreateAssetRequest,
  UpdateAssetRequest,
  ListAssetsParams,
  TransferAssetRequest,
  AssignAssetRequest,
  AddToCategoryRequest,
  AddPartsRequest,
  RemovePartsRequest,
  UpdateMaintenanceRequest,
  LendAssetRequest,
  CreateOTPRequest,
  OTPResponse,
} from '@23blocks/block-assets';
```
