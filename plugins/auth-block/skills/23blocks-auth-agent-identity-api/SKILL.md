---
name: 23blocks-auth-agent-identity-api
description: Authenticate AI agents with 23blocks APIs using AID (Agent Identity) protocol. Use when an agent needs to obtain its own Bearer token via Ed25519 keypair authentication instead of requiring a human-provided token.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Agent Identity (AID) Authentication API

Enable AI agents to authenticate themselves with any 23blocks API block using their own cryptographic identity. No human-provided Bearer tokens required.

## Overview

AID (Agent Identity) lets an AI agent:
1. Generate an Ed25519 keypair (via AMP)
2. Register its public key with a 23blocks tenant (one-time, admin-approved)
3. Exchange its identity for a JWT (per-session, automated)
4. Use the JWT as a standard Bearer token with any 23blocks API

The JWT works identically to a human-provided token -- all existing API skills work without modification.

## Prerequisites

- AMP identity initialized: `amp-init --auto`
- Admin must register the agent with the target tenant (one-time)
- `jq` and `openssl` available on the system

## Quick Start

```bash
# 1. Initialize AMP identity (first time only)
amp-init --auto

# 2. Register with a tenant (requires admin JWT)
aid-register.sh \
  --auth https://auth.api.us.23blocks.com/<tenant> \
  --token <admin-jwt> \
  --role-id <role-id>

# 3. Get a token (automated, cached)
export BLOCKS_AUTH_TOKEN=$(aid-token.sh \
  --auth https://auth.api.us.23blocks.com/<tenant> \
  --quiet)

# 4. Use any 23blocks API normally
curl -X GET "$BLOCKS_API_URL/posts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

## Authentication Flow

### Step 1: Identity Initialization

The agent needs an Ed25519 keypair. AMP provides this:

```bash
amp-init --auto
```

This creates:
- `~/.agent-messaging/agents/<name>/keys/private.pem` -- Private key (never shared)
- `~/.agent-messaging/agents/<name>/keys/public.pem` -- Public key
- `~/.agent-messaging/agents/<name>/config.json` -- Agent address, fingerprint

Check identity:
```bash
amp-identity.sh --json
```

Returns:
```json
{
  "agent_name": "my-agent",
  "address": "my-agent@tenant.aimaestro.local",
  "fingerprint": "SHA256:abc123...",
  "key_algorithm": "Ed25519"
}
```

### Step 2: Agent Registration (One-Time)

An admin registers the agent's public key with a 23blocks tenant. This links the agent to a role with specific permissions.

```bash
aid-register.sh \
  --auth https://auth.api.us.23blocks.com/<tenant> \
  --token <admin-jwt> \
  --role-id <role-id> \
  --api-key <api-key>
```

**Parameters:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `--auth` | Yes | Auth API URL with tenant path |
| `--token` | Yes | Admin JWT for authorization |
| `--role-id` | Yes | Role ID to assign (determines scopes) |
| `--api-key` | No | API key (X-Api-Key header) |
| `--name` | No | Display name (default: AMP agent name) |
| `--description` | No | Agent description |
| `--lifetime` | No | Token lifetime in seconds (default: 3600) |

**Curl equivalent:**
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
      "token_lifetime": 3600
    }
  }'
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "type": "agent_registration",
    "attributes": {
      "name": "my-agent",
      "amp_address": "my-agent@tenant.aimaestro.local",
      "status": "active",
      "role_id": 2,
      "token_lifetime": 3600
    }
  }
}
```

### Step 3: Token Exchange (Per-Session)

The agent requests a JWT by presenting its signed identity:

```bash
# Get just the token (for scripts)
TOKEN=$(aid-token.sh --auth https://auth.api.us.23blocks.com/<tenant> --quiet)

# Get full response as JSON
aid-token.sh --auth https://auth.api.us.23blocks.com/<tenant> --json

# Force fresh token (skip cache)
aid-token.sh --auth https://auth.api.us.23blocks.com/<tenant> --no-cache --quiet

# Request specific scopes
aid-token.sh --auth https://auth.api.us.23blocks.com/<tenant> --scope "files:read files:write" --quiet
```

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

**Token caching:** `aid-token.sh` automatically caches tokens at `~/.agent-messaging/tokens/` and reuses them until 60 seconds before expiry.

### Step 4: Use the Token

Set the standard environment variables and use any 23blocks API:

```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://content.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"

# Now any API call works
curl -X GET "$BLOCKS_API_URL/posts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Token Details

The AID JWT contains these claims:

| Claim | Example | Description |
|-------|---------|-------------|
| `sub` | `agent:uuid` | Subject (agent unique ID) |
| `token_type` | `agent` | Distinguishes from user tokens |
| `scopes` | `["files:read"]` | Permissions from assigned role |
| `agent_address` | `my-agent@...` | AMP address |
| `agent_name` | `my-agent` | Display name |
| `exp` | `1711234567` | Expiration timestamp |
| `iss` | `23blocks` | Issuer |

## Agent Registration Management

### Check Registration Status

```bash
aid-status.sh --auth https://auth.api.us.23blocks.com/<tenant>
```

### List Registrations (Admin)

```bash
curl -X GET "https://auth.api.us.23blocks.com/<tenant>/agent_registrations" \
  -H "Authorization: Bearer <admin-jwt>" \
  -H "Content-Type: application/json"
```

### Update Registration (Admin)

```bash
curl -X PUT "https://auth.api.us.23blocks.com/<tenant>/agent_registrations/<id>" \
  -H "Authorization: Bearer <admin-jwt>" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_registration": {
      "role_id": 3,
      "token_lifetime": 7200
    }
  }'
```

### Suspend/Reactivate Agent (Admin)

```bash
# Suspend
curl -X PUT "https://auth.api.us.23blocks.com/<tenant>/agent_registrations/<id>/suspend" \
  -H "Authorization: Bearer <admin-jwt>"

# Reactivate
curl -X PUT "https://auth.api.us.23blocks.com/<tenant>/agent_registrations/<id>/reactivate" \
  -H "Authorization: Bearer <admin-jwt>"
```

### Delete Registration (Admin)

```bash
curl -X DELETE "https://auth.api.us.23blocks.com/<tenant>/agent_registrations/<id>" \
  -H "Authorization: Bearer <admin-jwt>"
```

---

## Error Handling

| Error | HTTP | Cause | Fix |
|-------|------|-------|-----|
| `agent_not_registered` | 400 | Agent not registered with tenant | Run `aid-register.sh` |
| `invalid_grant` | 400 | Signature invalid or identity expired | Verify AMP keys match registration |
| `invalid_proof` | 400 | Proof of possession failed | Check system clock sync |
| `invalid_scope` | 400 | Requested scopes exceed permissions | Omit `--scope` to get all available |
| `agent_suspended` | 403 | Agent suspended by admin | Contact tenant admin |
| 401 | 401 | Registration token expired/invalid | Get fresh admin JWT |
| 422 | 422 | Agent already registered | Check existing registrations |

---

## Troubleshooting

**"AMP identity not initialized"**
```bash
amp-init --auto
```

**"Agent not registered"**
```bash
# Check if registered
aid-status.sh --auth https://auth.api.us.23blocks.com/<tenant>

# Register (need admin JWT)
aid-register.sh --auth https://auth.api.us.23blocks.com/<tenant> \
  --token <admin-jwt> --role-id <role-id>
```

**"Invalid proof -- clock skew"**
The proof timestamp must be within 300 seconds of the server. Sync your system clock:
```bash
# macOS
sudo sntp -sS time.apple.com

# Linux
sudo ntpdate pool.ntp.org
```

**"Token expired" during session**
```bash
# Force fresh token
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a <auth-url> --no-cache -q)
```

---

## Data Model

### AgentRegistration

| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Registration identifier |
| `name` | string | Agent display name |
| `amp_address` | string | Agent's AMP address |
| `amp_fingerprint` | string | SHA256 fingerprint of public key |
| `amp_public_key` | text | Ed25519 public key (PEM format) |
| `key_algorithm` | string | Always "Ed25519" |
| `role_id` | integer | Assigned role (determines scopes) |
| `token_lifetime` | integer | JWT lifetime in seconds |
| `status` | enum | active, suspended |
| `description` | string | Optional description |
| `created_at` | timestamp | Registration time |
| `updated_at` | timestamp | Last update |
