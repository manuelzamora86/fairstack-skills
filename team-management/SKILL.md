---
name: fairstack-team-management
description: |
  Manage your FairStack team via the API. Invite members by email, manage roles
  (owner/admin/member), view team info, handle invitations, and remove members.
  All team members share a single credit pool. Trigger on: team, invite member,
  team management, add teammate, team invite, org, organization, team settings,
  change role, remove member.
---

# Team Management

Manage your FairStack team — invite members, assign roles, and share a credit pool. All team members generate against a single balance.

**Base URL:** `https://fairstack.ai`

---

## Authentication

```bash
export FAIRSTACK_API_KEY="fs_live_..."
```

Get your API key at [fairstack.ai/app/api-keys](https://fairstack.ai/app/api-keys).

All team management endpoints require authentication. Role requirements noted per endpoint.

---

## Quick Start

### Get Team Info

```bash
curl https://fairstack.ai/v1/team \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

### CLI

```bash
# View your team
fairstack team info

# Invite a teammate
fairstack team invite --email alice@example.com --role member

# List pending invites
fairstack team invites

# List members
fairstack team members
```

---

## Endpoints

### Get Team Info

#### `GET /v1/team`

Returns your organization, members, and credit balance. **Any role** can call this.

```bash
curl https://fairstack.ai/v1/team \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

#### Response

```json
{
  "data": {
    "org": {
      "id": "org_abc123",
      "name": "My Team",
      "creditBalanceMicro": 50000000,
      "creditBalanceUsd": 50.00
    },
    "members": [
      {
        "userId": "user_001",
        "name": "Alice Smith",
        "email": "alice@example.com",
        "role": "owner",
        "joinedAt": "2026-01-15T10:00:00Z"
      },
      {
        "userId": "user_002",
        "name": "Bob Jones",
        "email": "bob@example.com",
        "role": "member",
        "joinedAt": "2026-03-20T14:30:00Z"
      }
    ],
    "pendingInvites": 1
  }
}
```

---

### Invite a Team Member

#### `POST /v1/team/invite`

Send an email invitation to join your team. Requires **admin** or **owner** role.

```bash
curl -X POST https://fairstack.ai/v1/team/invite \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "alice@example.com",
    "role": "member"
  }'
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | **Yes** | Email address to invite |
| `role` | string | **Yes** | `"admin"` or `"member"` (cannot invite as owner) |

#### Response (201)

```json
{
  "data": {
    "id": "inv_xyz789",
    "email": "alice@example.com",
    "role": "member",
    "status": "pending",
    "expiresAt": "2026-04-01T10:00:00Z",
    "createdAt": "2026-03-25T10:00:00Z"
  }
}
```

---

### List Pending Invites

#### `GET /v1/team/invites`

List all pending invitations. Requires **admin** or **owner** role.

```bash
curl https://fairstack.ai/v1/team/invites \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

#### Response

```json
{
  "data": {
    "invites": [
      {
        "id": "inv_xyz789",
        "email": "alice@example.com",
        "role": "member",
        "status": "pending",
        "expiresAt": "2026-04-01T10:00:00Z",
        "createdAt": "2026-03-25T10:00:00Z"
      }
    ]
  }
}
```

---

### Cancel an Invite

#### `DELETE /v1/team/invites/:id`

Cancel a pending invitation. Requires **admin** or **owner** role.

```bash
curl -X DELETE https://fairstack.ai/v1/team/invites/inv_xyz789 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

### Accept an Invite

#### `POST /v1/team/invites/:token/accept`

Accept an invitation using the token from the invite email. Requires authentication (the accepting user must be logged in).

```bash
curl -X POST https://fairstack.ai/v1/team/invites/abc123token/accept \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

#### Response

```json
{
  "data": {
    "orgId": "org_abc123",
    "orgName": "My Team",
    "role": "member"
  }
}
```

---

### Get Invite Info (Public)

#### `GET /v1/team/invites/:token/info`

Get details about an invite before accepting. **No authentication required.**

```bash
curl https://fairstack.ai/v1/team/invites/abc123token/info
```

#### Response

```json
{
  "data": {
    "orgName": "My Team",
    "inviterName": "Alice Smith",
    "role": "member",
    "email": "bob@example.com",
    "status": "pending",
    "expiresAt": "2026-04-01T10:00:00Z"
  }
}
```

---

### Change a Member's Role

#### `PATCH /v1/team/members/:userId`

Change a team member's role. Requires **owner** role.

```bash
curl -X PATCH https://fairstack.ai/v1/team/members/user_002 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"role": "admin"}'
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `role` | string | **Yes** | New role: `"admin"` or `"member"` |

---

### Remove a Member

#### `DELETE /v1/team/members/:userId`

Remove a member from the team. Requires **admin** or **owner** role. Cannot remove the owner.

```bash
curl -X DELETE https://fairstack.ai/v1/team/members/user_002 \
  -H "Authorization: Bearer $FAIRSTACK_API_KEY"
```

---

## JavaScript / TypeScript

```typescript
import FairStack from "@fairstack/sdk";

const fs = new FairStack({ apiKey: process.env.FAIRSTACK_API_KEY! });

// Get team info
const team = await fs.team.get();
console.log(`Team: ${team.org.name}`);
console.log(`Balance: $${team.org.creditBalanceUsd}`);
console.log(`Members: ${team.members.length}`);

// Invite a member
const invite = await fs.team.invite({
  email: "alice@example.com",
  role: "member",
});
console.log(`Invited: ${invite.email} (expires ${invite.expiresAt})`);

// List pending invites
const invites = await fs.team.invites();
console.log(`${invites.invites.length} pending invites`);

// Change role
await fs.team.changeRole("user_002", { role: "admin" });

// Remove member
await fs.team.removeMember("user_002");

// Cancel invite
await fs.team.cancelInvite("inv_xyz789");
```

## Python

```python
import requests, os

headers = {
    "Authorization": f"Bearer {os.environ['FAIRSTACK_API_KEY']}",
    "Content-Type": "application/json",
}

# Get team info
resp = requests.get("https://fairstack.ai/v1/team", headers=headers)
team = resp.json()["data"]
print(f"Team: {team['org']['name']}, Balance: ${team['org']['creditBalanceUsd']}")

# Invite a member
resp = requests.post(
    "https://fairstack.ai/v1/team/invite",
    headers=headers,
    json={"email": "alice@example.com", "role": "member"},
)
print(f"Invited: {resp.json()['data']['email']}")

# List pending invites
resp = requests.get("https://fairstack.ai/v1/team/invites", headers=headers)
invites = resp.json()["data"]["invites"]
print(f"{len(invites)} pending invites")

# Remove member
requests.delete(
    "https://fairstack.ai/v1/team/members/user_002",
    headers=headers,
)
```

---

## Roles

| Role | Permissions |
|------|-------------|
| **owner** | All permissions. Can change roles, transfer ownership. One per team. |
| **admin** | Invite/remove members, manage invites, view team info, generate. |
| **member** | View team info, generate. Cannot manage members or invites. |

---

## Shared Credit Pool

All team members share a single credit balance at the organization level.

- Credits are deducted from the team balance, not individual balances
- Any member can generate; the cost comes from the shared pool
- Only the owner or admin can add credits via the billing page
- Usage is tracked per-member for visibility in the usage dashboard

---

## Error Handling

| Status | Code | Meaning |
|--------|------|---------|
| 400 | — | Invalid email, cannot invite as owner, etc. |
| 403 | `FORBIDDEN` | Insufficient role (e.g., member trying to invite) |
| 404 | — | Invite not found, member not found, or expired |
| 409 | — | Email already a member, or invite already pending for that email |
| 429 | — | Rate limited |

---

## Best Practices

1. **Use admin sparingly.** Most teammates should be members. Only promote to admin when they need to manage invites or members.
2. **Monitor per-member usage.** Use the usage dashboard to see who is generating what and how much it costs.
3. **Set a spending cap.** Protect the shared pool with a monthly spending cap in billing settings.
