# Agent Identity (AID) - Endpoint Reference

## Token Exchange

### POST `/:company_url_id/oauth/token`

Exchange an Agent Identity for a JWT access token.

**Content-Type:** `application/x-www-form-urlencoded`

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `grant_type` | string | Yes | Must be `urn:aid:agent-identity` |
| `agent_identity` | string | Yes | Base64url-encoded signed Agent Identity JSON |
| `proof` | string | Yes | Base64url-encoded proof of possession |
| `scope` | string | No | Space-separated requested scopes |

**Agent Identity JSON structure (before base64url encoding):**

```json
{
  "aid_version": "1.0",
  "address": "my-agent@tenant.aimaestro.local",
  "alias": "my-agent",
  "public_key": "-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----",
  "key_algorithm": "Ed25519",
  "fingerprint": "SHA256:abc123...",
  "issued_at": "2026-03-22T00:00:00Z",
  "expires_at": "2026-09-22T00:00:00Z",
  "signature": "<base64-ed25519-signature-of-identity-without-signature-field>"
}
```

**Proof of Possession construction:**

1. Build the sign input string:
   ```
   aid-token-exchange\n<unix-timestamp>\n<auth-server-url>
   ```
2. Sign with Ed25519 private key
3. Concatenate: `signature_bytes + timestamp_string`
4. Base64url encode the result

**Proof max age:** 300 seconds (5 minutes)

**Curl example:**

```bash
# Build identity and proof (simplified - use aid-token.sh for production)
AGENT_IDENTITY_B64=$(echo -n "$SIGNED_IDENTITY_JSON" | base64 | tr '+/' '-_' | tr -d '=\n')
PROOF_B64=$(echo -n "$PROOF_BYTES" | base64 | tr '+/' '-_' | tr -d '=\n')

curl -X POST "https://auth.api.us.23blocks.com/<tenant>/oauth/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=urn%3Aaid%3Aagent-identity&agent_identity=${AGENT_IDENTITY_B64}&proof=${PROOF_B64}"
```

**Response (200 OK):**

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "files:read files:write content:read",
  "agent_address": "my-agent@tenant.aimaestro.local"
}
```

**Error Response (400):**

```json
{
  "error": "agent_not_registered",
  "error_description": "No active registration found for this agent identity"
}
```

---

## Agent Registration (Admin)

### POST `/:company_url_id/agent_registrations`

Register a new agent with the tenant.

**Authorization:** Bearer token (admin with `agent_registrations:write` scope)

```bash
curl -X POST "https://auth.api.us.23blocks.com/<tenant>/agent_registrations" \
  -H "Authorization: Bearer <admin-jwt>" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_registration": {
      "name": "my-agent",
      "amp_address": "my-agent@tenant.aimaestro.local",
      "amp_fingerprint": "SHA256:abc123...",
      "amp_public_key": "-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----",
      "key_algorithm": "Ed25519",
      "role_id": 2,
      "description": "Handles file management",
      "token_lifetime": 3600
    }
  }'
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "unique-id",
    "type": "agent_registration",
    "attributes": {
      "unique_id": "unique-id",
      "name": "my-agent",
      "amp_address": "my-agent@tenant.aimaestro.local",
      "amp_fingerprint": "SHA256:abc123...",
      "key_algorithm": "Ed25519",
      "role_id": 2,
      "status": "active",
      "description": "Handles file management",
      "token_lifetime": 3600,
      "created_at": "2026-03-22T00:00:00Z",
      "updated_at": "2026-03-22T00:00:00Z"
    }
  }
}
```

### GET `/:company_url_id/agent_registrations`

List all agent registrations.

**Authorization:** Bearer token (admin)

```bash
curl -X GET "https://auth.api.us.23blocks.com/<tenant>/agent_registrations" \
  -H "Authorization: Bearer <admin-jwt>" \
  -H "Content-Type: application/json"
```

### GET `/:company_url_id/agent_registrations/:id`

Get a specific agent registration.

```bash
curl -X GET "https://auth.api.us.23blocks.com/<tenant>/agent_registrations/<id>" \
  -H "Authorization: Bearer <admin-jwt>" \
  -H "Content-Type: application/json"
```

### PUT `/:company_url_id/agent_registrations/:id`

Update an agent registration.

```bash
curl -X PUT "https://auth.api.us.23blocks.com/<tenant>/agent_registrations/<id>" \
  -H "Authorization: Bearer <admin-jwt>" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_registration": {
      "role_id": 3,
      "token_lifetime": 7200,
      "description": "Updated description"
    }
  }'
```

### DELETE `/:company_url_id/agent_registrations/:id`

Delete an agent registration.

```bash
curl -X DELETE "https://auth.api.us.23blocks.com/<tenant>/agent_registrations/<id>" \
  -H "Authorization: Bearer <admin-jwt>"
```

### PUT `/:company_url_id/agent_registrations/:id/suspend`

Suspend an agent (revokes token exchange ability).

```bash
curl -X PUT "https://auth.api.us.23blocks.com/<tenant>/agent_registrations/<id>/suspend" \
  -H "Authorization: Bearer <admin-jwt>"
```

### PUT `/:company_url_id/agent_registrations/:id/reactivate`

Reactivate a suspended agent.

```bash
curl -X PUT "https://auth.api.us.23blocks.com/<tenant>/agent_registrations/<id>/reactivate" \
  -H "Authorization: Bearer <admin-jwt>"
```
