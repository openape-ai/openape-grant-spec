# 3. Grant Lifecycle

## Grant States

A grant transitions through the following states:

```
requested ──→ approved ──→ used
         │           │──→ expired
         │           └──→ revoked
         └──→ denied
```

| State | Description |
|-------|-------------|
| `requested` | Agent has submitted a grant request. Awaiting human decision. |
| `approved` | Human has approved the grant. Token is available. |
| `denied` | Human has denied the grant. |
| `used` | The grant has been consumed (applies to `allow_once`). |
| `expired` | The grant's time window has elapsed. |
| `revoked` | The grant was explicitly revoked by a human. |

## Grant Types

| Type | Behavior |
|------|----------|
| `allow_once` | Single use. Transitions to `used` after one successful execution. |
| `allow_ttl` | Valid for a specified duration. Multiple uses permitted until expiration. |
| `allow_always` | No expiration. Valid until explicitly revoked. |

Implementations SHOULD default to `allow_once` unless the approver explicitly selects a broader type. The `allow_always` type SHOULD require additional confirmation (e.g., a warning in the approval UI).

## Grant Request

### Prerequisites

The agent MUST authenticate with the Grants Server using a valid identity token obtained via DDISA. The Grants Server verifies the agent's identity before processing the request.

### Request Payload

```json
{
  "command": "apt install -y nginx",
  "reason": "Web server needed for deployment of project X",
  "cmd_hash": "sha256:a1b2c3d4e5f6...",
  "target": "server.example.com",
  "requested_type": "allow_once"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `command` | Yes | The exact command or operation to be authorized. |
| `reason` | Yes | Human-readable explanation of why the action is needed. |
| `cmd_hash` | Yes | `sha256(command)` — cryptographic binding to the exact command string. |
| `target` | No | The target system where the command will execute. |
| `requested_type` | No | Preferred grant type. Server/approver may override. |

### Response

```json
{
  "grant_id": "g_abc123",
  "status": "requested",
  "poll_url": "https://grants.example.com/grants/g_abc123",
  "ws_url": "wss://grants.example.com/grants/g_abc123/ws"
}
```

## Approval Flow

When a grant is requested, the Grants Server presents the request to an authorized human approver. The approval UI MUST display:

- The **exact command** to be executed
- The agent's **stated reason**
- The **requesting agent's identity**
- The **target system** (if specified)
- The **requested grant type**

The approver may:

1. **Approve** — optionally overriding the grant type or setting a custom TTL.
2. **Deny** — optionally providing a reason for denial.

### Dual Accountability: `decided_by`

The identity of the approver is recorded in the `decided_by` claim of the resulting authorization token. This creates dual accountability:

- The **agent** is accountable for requesting the action (`sub` + `act`).
- The **human** is accountable for approving it (`decided_by`).

## Polling and Webhooks

Agents retrieve grant decisions using one of two mechanisms:

### Polling

The agent periodically queries the `poll_url` returned in the grant request response.

```
GET /grants/g_abc123
Authorization: Bearer <identity-token>
```

Response includes the current `status`. Implementations SHOULD support `Retry-After` headers to guide polling intervals.

### WebSocket

The agent connects to the `ws_url` and receives a push notification when the grant status changes. This is RECOMMENDED for interactive use cases to minimize latency.

### Webhook (Server-to-Server)

For automated pipelines, the agent MAY register a webhook URL in the grant request. The Grants Server will POST the decision to that URL.
