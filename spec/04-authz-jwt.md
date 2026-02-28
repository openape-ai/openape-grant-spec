# 4. Authorization JWT

## Overview

When a grant is approved, the Grants Server issues an **Authorization JWT (AuthZ-JWT)** — a signed token that the agent presents to the target system as proof of authorization.

## Claims

| Claim | Required | Description |
|-------|----------|-------------|
| `sub` | Yes | Subject — the agent's identity (e.g., `agent@example.com`). |
| `act` | Yes | Actor — structured claim identifying the acting agent. |
| `iss` | Yes | Issuer — the Grants Server that issued this token. |
| `aud` | Yes | Audience — the target system that should accept this token. |
| `iat` | Yes | Issued at — Unix timestamp. |
| `exp` | Yes | Expiration — Unix timestamp. MUST be set for all grant types. |
| `grant_id` | Yes | Unique identifier of the grant. |
| `grant_type` | Yes | One of `allow_once`, `allow_ttl`, `allow_always`. |
| `cmd_hash` | Yes* | `sha256(command_string)` — binds the token to a specific command. |
| `request_hash` | Yes* | For proxy use: `sha256(METHOD + " " + URL + "\n" + BODY)`. |
| `decided_by` | Yes | Identity of the human who approved the grant. |
| `target` | No | Target system identifier. |

*Either `cmd_hash` or `request_hash` MUST be present. Both MAY be present.

## Command Hash (`cmd_hash`)

The `cmd_hash` binds the authorization token to an exact command string:

```
cmd_hash = sha256(command_string)
```

**Example:**

```
command_string = "apt install -y nginx"
cmd_hash = "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
```

The target system MUST compute `sha256(received_command)` and verify it matches the `cmd_hash` claim before executing.

## Request Hash (`request_hash`)

For HTTP proxy scenarios, the `request_hash` binds the token to a specific HTTP request:

```
request_hash = sha256(METHOD + " " + URL + "\n" + BODY)
```

**Example:**

```
input = "POST https://api.example.com/v1/deploy\n{\"version\":\"1.2.3\"}"
request_hash = "sha256:7d865e959b2466918c9863afca942d0fb89d7c9ac0c99bafc3749504ded97730"
```

## Example Token

### Header

```json
{
  "alg": "ES256",
  "typ": "JWT",
  "kid": "grants-key-2025"
}
```

### Payload

```json
{
  "sub": "agent@example.com",
  "act": {
    "sub": "agent-runtime-id-xyz"
  },
  "iss": "https://grants.example.com",
  "aud": "server.example.com",
  "iat": 1740700000,
  "exp": 1740700300,
  "grant_id": "g_abc123",
  "grant_type": "allow_once",
  "cmd_hash": "sha256:e3b0c44298fc1c149afbf4c8996fb924...",
  "decided_by": "admin@example.com",
  "target": "server.example.com"
}
```

## Verification

The target system (or proxy) MUST perform the following verification steps:

1. **Validate JWT signature** against the Grants Server's public key (obtained via JWKS endpoint).
2. **Check `exp`** — reject expired tokens.
3. **Check `aud`** — reject tokens not intended for this system.
4. **Verify `cmd_hash`** — compute `sha256(received_command)` and compare to the `cmd_hash` claim. Reject on mismatch.
5. **Verify `request_hash`** (if proxy) — compute `sha256(METHOD + " " + URL + "\n" + BODY)` from the actual request and compare. Reject on mismatch.
6. **Check `grant_type`** — for `allow_once`, verify the grant has not already been consumed (requires state tracking or coordination with the Grants Server).

**Critical:** The target system MUST perform hash verification locally. Relying solely on token validity without checking the hash defeats the command-binding guarantee.
