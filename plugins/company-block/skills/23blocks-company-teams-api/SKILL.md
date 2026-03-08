---
name: 23blocks-company-teams-api
description: Create and manage 23blocks company teams via REST API. Use when creating, updating, or deleting teams, managing team membership (join, remove, list members), or querying user team associations.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Teams API

Complete API reference for 23blocks company team management with membership.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Company API base URL | `https://company.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/teams" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/teams` | List teams with pagination |
| GET | `/teams/:unique_id` | Get team by ID |
| POST | `/teams` | Create new team |
| PUT | `/teams/:unique_id` | Update team |
| DELETE | `/teams/:unique_id` | Delete team |
| GET | `/teams/:unique_id/members` | List team members |
| POST | `/teams/:unique_id/join` | Add authenticated user to team |
| DELETE | `/teams/:unique_id/members/:user_unique_id` | Remove member from team |
| GET | `/users/:unique_id/teams` | List all teams a user belongs to |

---

## Data Models

### Team
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Team name |
| `description` | string | Team description |
| `department_id` | uuid | Associated department ID |
| `team_lead_id` | uuid | Team lead user ID |
| `status` | string | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### TeamMember
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Membership identifier |
| `user_unique_id` | uuid | User identifier |
| `email` | string | Member email |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `role` | string | Role within team (lead, member) |
| `joined_at` | timestamp | When member joined |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "409",
    "code": "conflict",
    "title": "Already a Member",
    "detail": "This user is already a member of the team."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-company
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
// Teams — client.company.teams
client.company.teams.list(params?: ListTeamsParams): Promise<PageResult<Team>>;
client.company.teams.get(uniqueId: string): Promise<Team>;
client.company.teams.create(data: CreateTeamRequest): Promise<Team>;
client.company.teams.update(uniqueId: string, data: UpdateTeamRequest): Promise<Team>;
client.company.teams.delete(uniqueId: string): Promise<void>;
client.company.teams.listByDepartment(departmentUniqueId: string): Promise<Team[]>;
```

### TypeScript Types

```typescript
import type {
  Team,
  CreateTeamRequest,
  UpdateTeamRequest,
  ListTeamsParams,
} from '@23blocks/block-company';
```

### React Hook

```typescript
import { useCompanyBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCompanyBlock();
  const result = await client.company.teams.list();
}
```
