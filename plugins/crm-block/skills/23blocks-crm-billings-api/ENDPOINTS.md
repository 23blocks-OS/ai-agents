# CRM Billings API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Endpoints

### Meeting Billing

#### GET /meetings/:meeting_unique_id/billing - Get Meeting Billing

Retrieves billing information for a specific meeting.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/meetings/meeting-uuid-123/billing" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "billing-uuid-123",
    "type": "billing",
    "attributes": {
      "unique_id": "billing-uuid-123",
      "meeting_id": "meeting-uuid-123",
      "amount": 250.00,
      "payer_name": "Acme Corp",
      "payer_email": "billing@acme.com",
      "billing_type": "session",
      "status": "pending",
      "created_at": "2026-01-10T10:30:00Z"
    }
  }
}
```

---

#### POST /meetings/:meeting_unique_id/billing - Create Meeting Billing

Creates a billing record for a meeting.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/meetings/meeting-uuid-123/billing" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "billing": {
      "amount": 250.00,
      "payer_name": "Acme Corp",
      "payer_email": "billing@acme.com",
      "billing_type": "session"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount` | decimal | Yes | Billing amount |
| `payer_name` | string | Yes | Payer name |
| `payer_email` | string | Yes | Payer email |
| `billing_type` | string | No | Billing type (session, consultation, etc.) |

**Response 201:**
```json
{
  "data": {
    "id": "billing-uuid-456",
    "type": "billing",
    "attributes": {
      "unique_id": "billing-uuid-456",
      "meeting_id": "meeting-uuid-123",
      "amount": 250.00,
      "payer_name": "Acme Corp",
      "payer_email": "billing@acme.com",
      "billing_type": "session",
      "status": "pending",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### Outstanding Payments

#### GET /billings/outstanding_by_payer - Outstanding by Payer

Retrieves outstanding billing amounts grouped by payer.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/billings/outstanding_by_payer" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "payer_name": "Acme Corp",
      "payer_email": "billing@acme.com",
      "total_outstanding": 1500.00,
      "billing_count": 6
    },
    {
      "payer_name": "Widget Inc",
      "payer_email": "finance@widget.com",
      "total_outstanding": 750.00,
      "billing_count": 3
    }
  ]
}
```

---

### EAP Sessions

#### GET /billings/eap_sessions/:participant_email/:payer_name - EAP Sessions

Retrieves EAP (Employee Assistance Program) session billing for a participant and payer combination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/billings/eap_sessions/jane@acme.com/Acme%20Corp" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "billing-uuid-789",
      "type": "billing",
      "attributes": {
        "unique_id": "billing-uuid-789",
        "participant_email": "jane@acme.com",
        "payer_name": "Acme Corp",
        "amount": 150.00,
        "session_date": "2026-02-10",
        "status": "paid"
      }
    }
  ],
  "meta": {
    "total_sessions": 4,
    "total_amount": 600.00
  }
}
```

---

### Reports

#### GET /billings/reports/revenue - Revenue Report

Generates a revenue report.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/billings/reports/revenue?start_date=2026-01-01&end_date=2026-03-31" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start_date` | date | No | Report start date |
| `end_date` | date | No | Report end date |

**Response 200:**
```json
{
  "data": {
    "total_revenue": 45000.00,
    "total_pending": 12000.00,
    "total_paid": 33000.00,
    "billing_count": 85,
    "period": {
      "start_date": "2026-01-01",
      "end_date": "2026-03-31"
    }
  }
}
```

---

#### GET /billings/reports/aging - Aging Report

Generates an aging report for outstanding billings.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/billings/reports/aging" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "current": 5000.00,
    "days_30": 3000.00,
    "days_60": 1500.00,
    "days_90": 800.00,
    "days_over_90": 200.00,
    "total_outstanding": 10500.00
  }
}
```

---

#### GET /billings/reports/participant/:participant_email - Participant Report

Generates a billing report for a specific participant.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/billings/reports/participant/jane@acme.com" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "participant_email": "jane@acme.com",
    "total_billed": 2400.00,
    "total_paid": 1800.00,
    "total_outstanding": 600.00,
    "sessions_count": 16,
    "billings": [
      {
        "id": "billing-uuid",
        "amount": 150.00,
        "status": "paid",
        "created_at": "2026-01-10T10:30:00Z"
      }
    ]
  }
}
```

---

### Billing CRUD

#### GET /billings/:unique_id - Get Billing

Retrieves a single billing record.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/billings/billing-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "billing-uuid-123",
    "type": "billing",
    "attributes": {
      "unique_id": "billing-uuid-123",
      "meeting_id": "meeting-uuid-123",
      "amount": 250.00,
      "payer_name": "Acme Corp",
      "payer_email": "billing@acme.com",
      "billing_type": "session",
      "status": "pending",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Billing not found

---

#### PUT /billings/:unique_id - Update Billing

Updates a billing record.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/billings/billing-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "billing": {
      "status": "paid",
      "amount": 275.00
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "billing-uuid-123",
    "type": "billing",
    "attributes": {
      "unique_id": "billing-uuid-123",
      "status": "paid",
      "amount": 275.00,
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

---

#### DELETE /billings/:unique_id - Delete Billing

Deletes a billing record.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/billings/billing-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### Payment Split

#### GET /billings/:unique_id/payment_split - Payment Split

Retrieves the payment split details for a billing record.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/billings/billing-uuid-123/payment_split" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "billing_id": "billing-uuid-123",
    "total_amount": 250.00,
    "splits": [
      {
        "recipient_name": "Dr. Smith",
        "recipient_email": "smith@clinic.com",
        "amount": 175.00,
        "percentage": 70
      },
      {
        "recipient_name": "Clinic Admin",
        "recipient_email": "admin@clinic.com",
        "amount": 75.00,
        "percentage": 30
      }
    ]
  }
}
```
