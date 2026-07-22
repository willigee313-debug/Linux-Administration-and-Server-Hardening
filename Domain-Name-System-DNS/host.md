# host

The `host` command is a simple, script-friendly DNS lookup utility used to convert hostnames to IP addresses and to query other DNS record types. It ships with the BIND `bind-utils` (RHEL/Fedora) or `bind9-dnsutils` (Debian/Ubuntu) package, alongside `dig` and `nslookup`, and is the quickest tool for one-line, human-readable answers.

## Overview

`host` performs forward and reverse DNS resolution against the resolvers listed in `/etc/resolv.conf`, or against a name server you name explicitly on the command line. Compared with its siblings, `host` favours brevity: a bare `host <domain>` returns the A, AAAA, and MX records in a single, plain-English block, making it ideal for quick sanity checks and shell scripts.

| Tool | Verbosity | Typical use |
| --- | --- | --- |
| `host` | Terse, one line per record | Fast lookups, scripting |
| `dig` | Structured, full DNS packet view | Debugging, zone transfers |
| `nslookup` | Interactive-capable, legacy | Ad-hoc multi-query sessions |

> [!NOTE]
> On a modern Linux host, `host` queries the stub resolver configured in `/etc/resolv.conf`. If `systemd-resolved` is active, that is usually `127.0.0.53`, which then forwards to your upstream DNS servers.

## Concepts

DNS resolution maps a human-readable name (`google.com`) to machine-usable data (an IP address, a mail server, etc.). Each answer is a **resource record** identified by a **type**:

| Record | Meaning | `host` flag |
| --- | --- | --- |
| A | IPv4 address | `-t a` (default) |
| AAAA | IPv6 address | `-t aaaa` |
| NS | Authoritative name server | `-t ns` |
| MX | Mail exchange server | `-t mx` |
| TXT | Free-form text (SPF, DKIM, verification) | `-t txt` |
| SOA | Start of Authority (zone metadata) | `-t soa` |
| PTR | Reverse pointer (IP → name) | auto for IP arguments |

A **forward** lookup resolves a name to an address; a **reverse** lookup resolves an address back to a name via a `PTR` record in the `in-addr.arpa` (IPv4) or `ip6.arpa` (IPv6) zone. `host` picks the direction automatically based on whether its argument parses as an IP address.

```mermaid
flowchart LR
    A[host google.com] --> B["Stub resolver<br/>/etc/resolv.conf"]
    B --> C[Recursive resolver]
    C --> D[Root / TLD / Authoritative]
    D --> C
    C --> B
    B --> A
```

## Options

The most commonly used flags:

| Flag | Effect |
| --- | --- |
| `-t <TYPE>` | Query a specific record type (`a`, `aaaa`, `ns`, `mx`, `txt`, `soa`, `any`) |
| `-a` | "All" — equivalent to `-t ANY -v`, a verbose dump of every record |
| `-v` | Verbose output (BIND `dig`-style formatting) |
| `-4` / `-6` | Use IPv4-only / IPv6-only transport to the resolver |
| `-r` | Non-recursive query (do not set the RD flag) |
| `-C` | Compare SOA records across a zone's authoritative name servers |
| `-W <sec>` | Wait at most `<sec>` seconds for a reply |

## Commands

### Look up a domain's IP address

Use `host` to find the IP address of a domain:

```bash
host google.com
```

### Reverse lookup (IP address to hostname)

Pass an IP address instead of a name; `host` issues a `PTR` query automatically:

```bash
host 8.8.8.8
```

### NS record (name server)

Query the name server for a domain:

```bash
host -t ns google.com
```

### MX record (mail exchange)

Query the mail exchange servers for a domain:

```bash
host -t mx google.com
```

### TXT record (text)

Query the TXT records for a domain:

```bash
host -t txt google.com
```

### SOA record (start of authority)

Query the SOA record for a domain:

```bash
host -t soa google.com
```

### Query the SOA record using a specific DNS server

Specify a particular DNS server to query the SOA record. Here the query is directed at `64.6.64.6` (a public Verisign resolver) instead of the system default:

```bash
host -t soa google.com 64.6.64.6
```

> [!TIP]
> Appending a resolver IP after the domain (`host <domain> <server>`) bypasses `/etc/resolv.conf`. This is invaluable for confirming that a change has propagated to a specific name server before it reaches your local cache.

## Examples

A bare lookup returns the common records together. Example (output abbreviated for illustration):

```text
$ host google.com
google.com has address 142.250.72.14
google.com has IPv6 address 2607:f8b0:4005:80a::200e
google.com mail is handled by 10 smtp.google.com.
```

A reverse lookup resolves an address back to its `PTR` name (output abbreviated):

```text
$ host 8.8.8.8
8.8.8.8.in-addr.arpa domain name pointer dns.google.
```

## Best Practices

- Use `host` for quick, readable checks; reach for `dig` when you need to inspect flags, TTLs, or the full response packet.
- Pin the resolver (`host <domain> <server>`) when verifying propagation across multiple authoritative or recursive servers.
- In scripts, prefer an explicit record type (`-t a`) so parsing stays deterministic across `host` versions.
- Use `host -C <zone>` to detect stale slaves — it flags any authoritative server serving an out-of-date SOA serial.

## Security Considerations

> [!WARNING]
> DNS queries issued by `host` are sent in **cleartext UDP/53** by default and can be observed or tampered with on-path. For confidentiality, front your resolvers with DNS-over-TLS (DoT) or DNS-over-HTTPS (DoH) at the system resolver, and validate answers with DNSSEC where the zone supports it.

- Treat TXT and SOA output as reconnaissance-relevant: SPF/DKIM records and admin contact fields in the SOA can leak infrastructure details useful to an attacker (this is why `host` appears in DNS enumeration workflows).
- Reverse (`PTR`) lookups can map an internal address range to descriptive hostnames; keep internal reverse zones off public resolvers.
- Do not rely on unauthenticated DNS answers for security decisions; a spoofed or cache-poisoned response can redirect traffic.

## Troubleshooting

| Symptom | Likely cause | Action |
| --- | --- | --- |
| `Host not found (NXDOMAIN)` | Name does not exist | Verify spelling; try a different resolver |
| `connection timed out; no servers could be reached` | Resolver unreachable / firewall | Check `/etc/resolv.conf`, test port 53 reachability |
| `SERVFAIL` | Resolver or upstream failure, DNSSEC validation failure | Query authoritative server directly with `host <domain> <ns>` |
| `3(NXDOMAIN)` on a reverse lookup | No `PTR` record published for the address | Expected for many hosts; confirm the reverse zone is delegated |

> [!NOTE]
> **📸 Screenshot**
> _Capture: Terminal showing `host google.com` returning the A, AAAA, and MX records in a single block_

## References

- `man 1 host` — command reference
- RFC 1035 — Domain Names: Implementation and Specification
- BIND 9 Administrator Reference Manual (ISC)

## Related

- [dig](dig.md) — sibling DNS client tool with full packet-level output
- [nslookup](nslookup.md) — sibling interactive DNS client tool
- [Dns-Client-Tools](Dns-Client-Tools.md) — parent overview of DNS client utilities
- [DNS-Server-Types](DNS-Server-Types.md) — how authoritative, recursive, and caching servers differ
- [Reverse-Zone](Reverse-Zone.md) — configuring the reverse (`in-addr.arpa`) zone that answers `PTR` lookups
- [Readme](../Readme.md) — Domain Name System (DNS) module index
- DNS-Enumeration — using host for DNS reconnaissance and enumeration
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
