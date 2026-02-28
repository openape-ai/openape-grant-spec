# 1. Overview

## Problem

AI agents require elevated privileges to perform useful work — installing packages, modifying infrastructure, calling restricted APIs. However, granting agents permanent administrative rights creates unacceptable risk:

- A compromised agent can execute arbitrary privileged commands.
- A misbehaving agent can cause irreversible damage without human awareness.
- There is no audit trail of *why* a privileged action was taken.

Static permission models (role-based, capability-based) are insufficient because they cannot capture the dynamic, context-dependent nature of agent operations.

## Solution

OpenApe Grants defines a protocol for **scoped, time-limited authorization grants** with explicit human approval. The core flow is:

1. **Agent requests a grant** — specifying the exact command, a human-readable reason, and a cryptographic hash of the command.
2. **Human reviews and decides** — the approver sees both the command and the reason, then approves or denies.
3. **Agent receives an authorization token** — a signed JWT binding the grant to the specific command.
4. **Target system verifies and executes** — the executing system validates the token and command hash before proceeding.

This ensures that no privileged action occurs without explicit human consent, while maintaining a complete audit trail.

## Design Principles

### Deny by Default

Agents have no implicit privileges. Every privileged action requires an explicit grant. If no grant exists, the action MUST be denied.

### Minimal Privilege

Grants are scoped to the narrowest possible permission:

- A specific command (bound by `cmd_hash`)
- A specific time window (`exp` claim)
- A specific target system (`aud` / `target` claim)

### Auditability

Every grant produces a verifiable record:

- Who requested it (`sub`, `act`)
- What was requested (`cmd_hash`, command)
- Who approved it (`decided_by`)
- When it was issued and when it expires (`iat`, `exp`)

### Replay Protection

Grants include mechanisms to prevent reuse:

- `once` grants are single-use
- All grants carry expiration times
- Command hashes bind tokens to exact operations
