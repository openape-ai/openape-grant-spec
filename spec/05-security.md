# 5. Security Considerations

## Threat Model

### 5.1 Approval Fatigue

**Threat:** Frequent grant requests desensitize human approvers, leading to rubber-stamp approvals of malicious or unnecessary actions.

**Mitigations:**

- **Rate limiting** — the Grants Server SHOULD enforce per-agent rate limits on grant requests.
- **Risk scoring** — requests SHOULD be classified by risk level (e.g., read-only vs. destructive). High-risk requests SHOULD require additional friction (e.g., confirmation dialog, delay).
- **Batching controls** — agents SHOULD NOT be permitted to submit large numbers of requests in rapid succession.
- **Approval summaries** — the UI SHOULD show approvers their recent approval history to surface patterns.

### 5.2 Command Substitution

**Threat:** An agent requests approval for a benign command but executes a different, malicious command.

**Mitigations:**

- **`cmd_hash` binding** — the AuthZ-JWT contains a hash of the exact approved command. The target system MUST verify the hash against the actual command before execution.
- This is the primary defense and MUST NOT be bypassed.

### 5.3 Reason vs. Command Mismatch

**Threat:** An agent provides a plausible reason that does not match the actual command (e.g., reason says "install web server" but command is `rm -rf /`).

**Mitigations:**

- **UI MUST display both** — the approval interface MUST show the exact command alongside the stated reason. Approvers MUST be able to see both before deciding.
- **Command highlighting** — destructive or unusual commands SHOULD be visually highlighted.

### 5.4 Replay Attacks

**Threat:** A valid AuthZ-JWT is captured and reused to execute the same command again.

**Mitigations:**

- **`allow_once` grants** — single-use tokens that are invalidated after first use.
- **Short TTLs** — `allow_ttl` grants SHOULD use the shortest practical expiration.
- **Nonce tracking** — target systems MAY track used `grant_id` values to prevent replay.

### 5.5 Agent-compiled Binaries

**Threat:** An agent compiles and installs a binary that contains malicious code. The grant request shows source compilation commands, but the resulting binary is opaque.

**Mitigations:**

- **Policy: CI/CD only** — organizations SHOULD prohibit agents from compiling and installing binaries directly. All builds SHOULD go through auditable CI/CD pipelines.
- **Never install agent-built binaries** on production systems. This SHOULD be enforced by policy and, where possible, by technical controls (e.g., allow-listing package sources).

### 5.6 Staged File Attacks

**Threat:** An agent writes malicious content to a file in one step (no grant required for file writes), then requests a grant to execute or source that file in a separate step. The approved command appears benign (e.g., `bash setup.sh`), but the file content is malicious.

**Status:** This is an **accepted residual risk** in the current protocol. Grants bind to the command, not to the full filesystem state.

**Partial mitigations:**

- Auditing of file modifications alongside grant requests.
- Execution policies that restrict which files agents may create or modify.
- Sandboxed execution environments that limit blast radius.

## Audit Requirements

Implementations MUST maintain an audit log containing:

- All grant requests (including denied ones)
- The full command and reason for each request
- The identity of the requester and the approver
- Timestamps for each state transition
- The resulting AuthZ-JWT (or a reference to it)

Audit logs MUST be tamper-resistant. Implementations SHOULD use append-only storage or cryptographic chaining.

Audit logs MUST be retained for a minimum period defined by organizational policy (RECOMMENDED: at least 90 days).

## Key Management

- The Grants Server MUST sign AuthZ-JWTs using asymmetric keys (RECOMMENDED: ES256 or EdDSA).
- Public keys MUST be available via a JWKS endpoint at a well-known URL relative to the Grants Server (e.g., `<grants-server-url>/.well-known/jwks.json`).
- Keys SHOULD be rotated periodically (RECOMMENDED: at least every 90 days).
- Revoked keys MUST be removed from the JWKS endpoint.
- Key rotation MUST NOT invalidate unexpired tokens (maintain old keys in JWKS until all tokens signed with them have expired).
