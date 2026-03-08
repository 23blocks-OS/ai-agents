# Addresses API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Endpoints

### GET /addresses/:unique_id/ - Get Address

Retrieves a single address by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/addresses/addr-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "addr-uuid-123",
    "type": "address",
    "attributes": {
      "unique_id": "addr-uuid-123",
      "street": "456 Oak Ave",
      "city": "Los Angeles",
      "state": "CA",
      "zip_code": "90001",
      "country": "US",
      "latitude": 33.9425,
      "longitude": -118.2551,
      "address_type": "home",
      "is_default": true,
      "owner_type": "user",
      "owner_id": "user-uuid-123",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "tags": {
        "data": [
          { "id": "tag-uuid", "type": "tag" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Address not found

---

### GET /addresses/owned_by/:unique_id - Get Addresses by Owner

Retrieves all addresses owned by a specific entity.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/addresses/owned_by/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "addr-uuid-123",
      "type": "address",
      "attributes": {
        "unique_id": "addr-uuid-123",
        "street": "456 Oak Ave",
        "city": "Los Angeles",
        "state": "CA",
        "zip_code": "90001",
        "country": "US",
        "address_type": "home",
        "is_default": true,
        "owner_type": "user",
        "owner_id": "user-uuid-123"
      }
    }
  ]
}
```

---

### POST /addresses/ - Create Address

Creates a new address.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/addresses/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "address": {
      "street": "456 Oak Ave",
      "city": "Los Angeles",
      "state": "CA",
      "zip_code": "90001",
      "country": "US",
      "latitude": 33.9425,
      "longitude": -118.2551,
      "address_type": "home",
      "is_default": true,
      "owner_type": "user",
      "owner_id": "user-uuid-123"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `street` | string | Yes | Street address |
| `city` | string | Yes | City name |
| `state` | string | Yes | State/province code |
| `zip_code` | string | Yes | Postal/zip code |
| `country` | string | Yes | Country code (ISO 3166) |
| `latitude` | float | No | Latitude coordinate |
| `longitude` | float | No | Longitude coordinate |
| `address_type` | string | No | Type (home, work, billing, shipping) |
| `is_default` | boolean | No | Whether this is the default address |
| `owner_type` | string | Yes | Owner entity type (user, contact) |
| `owner_id` | uuid | Yes | Owner entity unique ID |

**Response 201:**
```json
{
  "data": {
    "id": "addr-uuid-123",
    "type": "address",
    "attributes": {
      "unique_id": "addr-uuid-123",
      "street": "456 Oak Ave",
      "city": "Los Angeles",
      "state": "CA",
      "zip_code": "90001",
      "country": "US",
      "latitude": 33.9425,
      "longitude": -118.2551,
      "address_type": "home",
      "is_default": true,
      "owner_type": "user",
      "owner_id": "user-uuid-123",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /addresses/:unique_id - Update Address

Updates an existing address.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/addresses/addr-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "address": {
      "street": "789 Pine St",
      "zip_code": "90002",
      "is_default": false
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "addr-uuid-123",
    "type": "address",
    "attributes": {
      "unique_id": "addr-uuid-123",
      "street": "789 Pine St",
      "city": "Los Angeles",
      "state": "CA",
      "zip_code": "90002",
      "country": "US",
      "is_default": false,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Address not found

---

### DELETE /addresses/:unique_id - Delete Address

Deletes an address.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/addresses/addr-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Address not found

---

## Address Tags

### POST /addresses/:unique_id/tags/ - Add Tag

Adds a tag to an address.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/addresses/addr-uuid-123/tags/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag_unique_id": "tag-uuid-456"
  }'
```

**Response 200:**
```json
{
  "message": "Tag added successfully"
}
```

---

### DELETE /addresses/:unique_id/tags/:tag_unique_id - Remove Tag

Removes a tag from an address.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/addresses/addr-uuid-123/tags/tag-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Tag removed successfully"
}
```

---

## User Addresses

### GET /users/:unique_id/addresses - User Addresses

Retrieves all addresses for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/addresses" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "addr-uuid-123",
      "type": "address",
      "attributes": {
        "unique_id": "addr-uuid-123",
        "street": "456 Oak Ave",
        "city": "Los Angeles",
        "state": "CA",
        "zip_code": "90001",
        "country": "US",
        "address_type": "home",
        "is_default": true
      }
    },
    {
      "id": "addr-uuid-456",
      "type": "address",
      "attributes": {
        "unique_id": "addr-uuid-456",
        "street": "100 Corporate Blvd",
        "city": "Los Angeles",
        "state": "CA",
        "zip_code": "90010",
        "country": "US",
        "address_type": "work",
        "is_default": false
      }
    }
  ]
}
```

---

### GET /users/:unique_id/default - User Default Address

Retrieves the default address for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/default" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "addr-uuid-123",
    "type": "address",
    "attributes": {
      "unique_id": "addr-uuid-123",
      "street": "456 Oak Ave",
      "city": "Los Angeles",
      "state": "CA",
      "zip_code": "90001",
      "country": "US",
      "address_type": "home",
      "is_default": true
    }
  }
}
```

**Errors:**
- `404 Not Found` - No default address set

---

## Contact Addresses

### GET /contacts/:unique_id/addresses - Contact Addresses

Retrieves all addresses for a contact.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/contacts/contact-uuid-789/addresses" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "addr-uuid-789",
      "type": "address",
      "attributes": {
        "unique_id": "addr-uuid-789",
        "street": "200 Business Park Dr",
        "city": "San Francisco",
        "state": "CA",
        "zip_code": "94105",
        "country": "US",
        "address_type": "work",
        "is_default": true
      }
    }
  ]
}
```
