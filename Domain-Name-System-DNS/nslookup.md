# nslookup

The `nslookup` (Name Server Lookup) command is a network administration tool for querying the Domain Name System (DNS) to obtain domain-name-to-IP-address mappings and other DNS records. It supports both a single-shot command mode and an interactive session mode.

## Overview

`nslookup` ships with the BIND utilities (`bind-utils` on RHEL/Fedora, `bind9-dnsutils` on Debian/Ubuntu) alongside `dig` and `host`. It resolves names against the servers in `/etc/resolv.conf`, or against a name server you specify. Although `dig` is generally preferred for scripting and debugging, `nslookup` remains widely used because of its cross-platform availability (it behaves almost identically on Windows) and its convenient interactive mode.

| Tool | Strength | Notes |
| --- | --- | --- |
| `nslookup` | Interactive session, cross-platform | Legacy but ubiquitous |
| `dig` | Full DNS packet inspection | Preferred for debugging/scripts |
| `host` | Terse one-line answers | Fast checks and scripting |

> [!NOTE]
> On systems using `systemd-resolved`, the default resolver in `/etc/resolv.conf` is often `127.0.0.53`, which forwards to your configured upstream servers.

## Concepts

Each DNS answer is a **resource record** identified by a **type**. `nslookup` selects the type with `-type=<TYPE>` in command mode or `set type=<TYPE>` in interactive mode.

| Record | Meaning |
| --- | --- |
| A | IPv4 address |
| AAAA | IPv6 address |
| NS | Authoritative name server |
| MX | Mail exchange server |
| SOA | Start of Authority (zone metadata) |
| TXT | Free-form text (SPF, DKIM, verification) |
| ANY | All available record types |

```mermaid
flowchart LR
    A[nslookup google.com] --> B["Stub resolver<br/>/etc/resolv.conf"]
    B --> C[Recursive resolver]
    C --> D[Root / TLD / Authoritative]
    D --> C
    C --> B
    B --> A
```

## Options

Command-mode options are passed with a leading `-`; the same switches are available inside interactive mode with the `set` keyword (for example, `-type=NS` becomes `set type=ns`).

| Command mode | Interactive equivalent | Effect |
| --- | --- | --- |
| `-type=<TYPE>` | `set type=<TYPE>` | Select the record type to query |
| `-port=<N>` | `set port=<N>` | Query a non-standard DNS port |
| `-timeout=<sec>` | `set timeout=<sec>` | Set the per-query wait time |
| `-debug` | `set debug` | Show the full response packet |
| `-norecurse` | `set norecurse` | Send a non-recursive query (clear RD flag) |
| `<domain> <server>` | `server <ip>` | Direct queries at a specific name server |

## Commands

### Look up a domain's IP address

Use `nslookup` to find the IP address of a domain:

```bash
nslookup google.com
```

### Reverse lookup (IP address to hostname)

Pass an IP address to resolve it back to a hostname via its `PTR` record:

```bash
nslookup 8.8.8.8
```

### Look up a domain using a specific DNS server

Query a domain using a specific DNS server (here, the public Verisign resolver `64.6.64.6`):

```bash
nslookup google.com 64.6.64.6
```

### A record (IPv4)

Query the A record (IPv4 address) of a domain:

```bash
nslookup -type=A google.com
```

### AAAA record (IPv6)

Query the AAAA record (IPv6 address) of a domain:

```bash
nslookup -type=AAAA google.com
```

### NS record (name server)

Query the name server for a domain:

```bash
nslookup -type=NS google.com
```

### MX record (mail exchange)

Query the mail exchange servers for a domain:

```bash
nslookup -type=MX google.com
```

### SOA record (start of authority)

Query the SOA record for a domain:

```bash
nslookup -type=SOA google.com
```

### TXT record (text)

Query the TXT records for a domain:

```bash
nslookup -type=TXT google.com
```

### Query all available records

Retrieve all available DNS records for a domain:

```bash
nslookup -type=ANY google.com
```

> [!TIP]
> Many recursive resolvers now refuse or truncate `ANY` queries (RFC 8482) to reduce amplification abuse. If `-type=ANY` returns little, enumerate the specific record types you need instead.

## Interactive Mode

You can use `nslookup` in interactive mode to perform multiple queries without reopening the tool each time.

### Start interactive mode

Launch `nslookup` without any arguments:

```bash
nslookup
```

### Example interactive commands

1. Query a specific domain:

```text
facebook.com
```

2. Set the query type to NS records, then query:

```text
set type=ns
```

```text
facebook.com
```

3. Set the query type to TXT records, then query:

```text
set type=txt
```

```text
facebook.com
```

4. Use a specific DNS server for subsequent queries:

```text
server 64.6.64.6
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: nslookup interactive session showing `set type=ns` followed by a facebook.com query returning name server records_

## Best Practices

- Prefer `dig` for scripting and precise debugging; use `nslookup` interactive mode for exploratory, multi-record sessions.
- Pin the resolver with `server <ip>` (interactive) or a trailing IP (command mode) when verifying DNS propagation across servers.
- Query specific record types rather than relying on `ANY`, which is increasingly rate-limited or refused.

## Security Considerations

> [!WARNING]
> `nslookup` sends queries in **cleartext UDP/53** by default; they can be observed or spoofed on-path. Protect confidentiality with DNS-over-TLS/HTTPS at the system resolver and validate answers with DNSSEC where available.

- SOA, NS, and TXT output can reveal infrastructure details (admin contacts, mail providers, SPF/DKIM setup) useful for reconnaissance — this is why `nslookup` features in DNS enumeration and zone-transfer testing.
- Never make security decisions based on unauthenticated DNS answers, which can be forged or cache-poisoned.

## Troubleshooting

| Symptom | Likely cause | Action |
| --- | --- | --- |
| `** server can't find <domain>: NXDOMAIN` | Name does not exist | Verify spelling; try another resolver |
| `;; connection timed out; no servers could be reached` | Resolver unreachable / firewall | Check `/etc/resolv.conf` and port 53 reachability |
| `** server can't find <domain>: SERVFAIL` | Upstream failure or DNSSEC validation error | Query the authoritative server directly |
| Sparse or empty `-type=ANY` result | Resolver refuses ANY (RFC 8482) | Query individual record types |

## References

- `man 1 nslookup` — command reference
- RFC 8482 — Providing Minimal-Sized Responses to DNS Queries That Have QTYPE=ANY
- BIND 9 Administrator Reference Manual (ISC)

## Related

- [dig](dig.md) — sibling DNS client tool with full packet-level output
- [host](host.md) — sibling terse DNS client tool
- [Dns-Client-Tools](Dns-Client-Tools.md) — parent overview of DNS client utilities
- [DNS-Server-Types](DNS-Server-Types.md) — authoritative, recursive, and caching server roles
- [Reverse-Zone](Reverse-Zone.md) — the reverse (`in-addr.arpa`) zone that answers `PTR` lookups
- [Readme](../Readme.md) — Domain Name System (DNS) module index
- DNS-Enumeration — nslookup used in DNS enumeration and zone-transfer tests
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
