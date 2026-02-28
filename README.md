# OpenApe Grants — Grant-based Authorization Protocol for AI Agents

**Status:** Draft

## Motivation

AI agents increasingly operate on behalf of humans — executing commands, accessing APIs, and managing infrastructure. Granting these agents permanent elevated privileges is dangerous: a compromised or misbehaving agent with admin rights can cause irreversible damage.

OpenApe Grants introduces a **human-in-the-loop approval system** for privileged actions. Instead of blanket permissions, agents request scoped, time-limited grants that require explicit human authorization before execution. This keeps humans in control while enabling agents to operate effectively.

## Overview

The protocol defines:

- **Grant lifecycle** — how agents request, humans approve, and systems enforce authorization grants
- **DNS discovery** — how agents locate the Grants Server for a given domain
- **Authorization JWTs** — cryptographically signed tokens binding a grant to a specific command
- **Security model** — threat analysis and mitigations for agent-specific attack vectors

## Specification

| Document | Description |
|----------|-------------|
| [01 — Overview](spec/01-overview.md) | Problem statement, solution, and design principles |
| [02 — DNS Discovery](spec/02-dns-discovery.md) | Discovering the Grants Server via DNS |
| [03 — Grant Lifecycle](spec/03-grant-lifecycle.md) | Grant states, types, request and approval flow |
| [04 — AuthZ JWT](spec/04-authz-jwt.md) | Authorization token format and verification |
| [05 — Security](spec/05-security.md) | Threat model, audit, and key management |
| [06 — Use Cases](spec/06-use-cases.md) | Practical application scenarios |

## Related Specifications

This protocol builds on [DDISA (DNS-Discoverable Identity Service Architecture)](https://github.com/openape-ai/ddisa-spec), which provides the identity and authentication layer that OpenApe Grants extends with authorization.

## Links

- Website: [openape.at](https://openape.at)
- DDISA Spec: [github.com/openape-ai/ddisa-spec](https://github.com/openape-ai/ddisa-spec)

## License

This specification is published under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
