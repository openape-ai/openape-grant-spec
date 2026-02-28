# 2. DNS Discovery

## Overview

Agents discover the Grants Server for a given domain using DNS TXT records, following the same pattern established by [DDISA](https://github.com/openape-ai/ddisa-spec) for identity provider discovery.

## DNS Record Format

```
_openape_grants.<domain>. IN TXT "v=openape1 url=<grants-server-url>"
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `v`   | Yes      | Protocol version. MUST be `openape1`. |
| `url` | Yes      | HTTPS URL of the Grants Server endpoint. |

### Example

```
_openape_grants.example.com. IN TXT "v=openape1 url=https://grants.example.com"
```

## Resolution Process

1. Extract the domain from the agent's authenticated identity (e.g., `agent@example.com` → `example.com`).
2. Query DNS for `_openape_grants.example.com TXT`.
3. Parse the TXT record and extract the `url` value.
4. Use the URL as the Grants Server endpoint.

If no `_openape_grants` record exists, the domain does not support OpenApe Grants. The agent MUST NOT fall back to unauthenticated operation.

## Relationship with DDISA Discovery

A domain typically publishes two discovery records:

| Record | Purpose |
|--------|---------|
| `_ddisa.<domain>` | Identity Provider (authentication) |
| `_openape_grants.<domain>` | Grants Server (authorization) |

These MAY point to the same server or to different servers. The identity provider handles authentication (proving *who* the agent is), while the Grants Server handles authorization (determining *what* the agent may do).

### Example: Separate Servers

```
_ddisa.example.com.            IN TXT "v=ddisa1 url=https://id.example.com"
_openape_grants.example.com.   IN TXT "v=openape1 url=https://grants.example.com"
```

### Example: Combined Server

```
_ddisa.example.com.            IN TXT "v=ddisa1 url=https://openape.example.com"
_openape_grants.example.com.   IN TXT "v=openape1 url=https://openape.example.com"
```

## Security Considerations

- DNS lookups SHOULD use DNSSEC where available.
- Clients MUST only connect to Grants Server URLs using HTTPS.
- Clients SHOULD cache DNS results respecting TTL values to avoid repeated lookups.
