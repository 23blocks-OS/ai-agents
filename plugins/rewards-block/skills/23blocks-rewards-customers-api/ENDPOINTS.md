# Customers API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Endpoints

### GET /customers - List Customers

Lists all reward customers with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

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
      "id": "customer-uuid-123",
      "type": "customer",
      "attributes": {
        "unique_id": "customer-uuid-123",
        "name": "Jane Smith",
        "email": "jane@example.com",
        "total_points": 1250,
        "tier": "gold",
        "lifetime_points": 5400,
        "joined_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 10,
    "totalRecords": 148
  }
}
```

---

### GET /customers/:unique_id/ - Get Customer

Retrieves a single customer reward profile.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "customer-uuid-123",
    "type": "customer",
    "attributes": {
      "unique_id": "customer-uuid-123",
      "name": "Jane Smith",
      "email": "jane@example.com",
      "total_points": 1250,
      "tier": "gold",
      "lifetime_points": 5400,
      "joined_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Customer not found

---

### GET /customers/:unique_id/loyalty - Customer Loyalty Status

Retrieves the customer's current loyalty program status and tier information.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/loyalty" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "customer-uuid-123",
    "type": "customer_loyalty",
    "attributes": {
      "customer_unique_id": "customer-uuid-123",
      "loyalty_unique_id": "loyalty-uuid-1",
      "loyalty_name": "Gold Rewards",
      "current_tier": "gold",
      "total_points": 1250,
      "points_to_next_tier": 750,
      "next_tier": "platinum",
      "tier_expiry": "2026-12-31T23:59:59Z",
      "enrolled_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Customer not found or not enrolled in loyalty program

---

### GET /customers/:unique_id/rewards - Customer Rewards

Retrieves the customer's current reward balances.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/rewards?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

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
      "id": "reward-uuid-1",
      "type": "reward",
      "attributes": {
        "unique_id": "reward-uuid-1",
        "customer_id": "customer-uuid-123",
        "points": 75,
        "reward_type": "purchase",
        "reference_id": "order-12345",
        "description": "Points earned for order #12345",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
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

### GET /customers/:unique_id/rewards/expirations - Expiring Rewards

Retrieves the customer's rewards that are approaching expiration.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/rewards/expirations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "reward-uuid-2",
      "type": "reward_expiration",
      "attributes": {
        "unique_id": "reward-uuid-2",
        "points": 200,
        "expires_at": "2026-03-15T23:59:59Z",
        "days_remaining": 24,
        "reward_type": "purchase",
        "description": "Points from holiday purchases"
      }
    }
  ]
}
```

---

### GET /customers/:unique_id/rewards/history - Reward History

Retrieves the customer's complete reward transaction history.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/rewards/history?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

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
      "id": "history-uuid-1",
      "type": "reward_history",
      "attributes": {
        "unique_id": "history-uuid-1",
        "action": "earn",
        "points": 75,
        "balance_after": 1250,
        "reward_type": "purchase",
        "reference_id": "order-12345",
        "description": "Points earned for order #12345",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "history-uuid-2",
      "type": "reward_history",
      "attributes": {
        "unique_id": "history-uuid-2",
        "action": "redeem",
        "points": -500,
        "balance_after": 1175,
        "reward_type": "redemption",
        "reference_id": "redemption-456",
        "description": "Points redeemed for gift card",
        "created_at": "2025-01-08T14:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 68
  }
}
```

---

### GET /customers/:unique_id/badges - Customer Badges

Retrieves all badges earned by the customer.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/badges" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "award-uuid-1",
      "type": "badge_award",
      "attributes": {
        "unique_id": "award-uuid-1",
        "badge_unique_id": "badge-uuid-123",
        "badge_name": "First Purchase",
        "badge_description": "Awarded after first purchase",
        "badge_image_url": "https://cdn.example.com/badges/first-purchase.png",
        "points_awarded": 50,
        "awarded_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /customers/:unique_id/coupons - Customer Coupons

Retrieves all coupons assigned to or redeemed by the customer.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/coupons" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "coupon-uuid-1",
      "type": "customer_coupon",
      "attributes": {
        "unique_id": "coupon-uuid-1",
        "code": "SUMMER-ABC123",
        "discount_type": "percentage",
        "discount_value": 20,
        "status": "redeemed",
        "redeemed_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /customers/:unique_id/offer_codes - Customer Offer Codes

Retrieves all offer codes used by the customer.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/offer_codes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "offer-uuid-1",
      "type": "customer_offer_code",
      "attributes": {
        "unique_id": "offer-uuid-1",
        "code": "WELCOME50",
        "offer_type": "discount",
        "discount_value": 50,
        "status": "redeemed",
        "redeemed_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```
