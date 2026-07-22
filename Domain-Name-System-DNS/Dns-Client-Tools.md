# Dns Client Tools

DNS client tools let you query nameservers, validate zone data, and troubleshoot resolution from the command line. On Red Hat–family systems these ship in the **`bind-utils`** package (`dig`, `host`, `nslookup`, and friends). This note covers installing and inspecting the package, the tools it provides, and the DNS record types you will query with them.

## Overview

| Tool | Purpose |
| --- | --- |
| `dig` | Flexible, scriptable DNS lookups — the tool of choice for diagnostics |
| `host` | Simple, human-friendly lookups |
| `nslookup` | Legacy interactive/one-shot lookups |
| `delv` | DNS lookup with DNSSEC validation |
| `mdig` | Batch/multiple-query version of `dig` |
| `nsupdate` | Dynamic DNS record updates |

## Commands

### Check installed BIND packages

Use the following command to list installed BIND packages:

```bash
rpm -qa | grep bind
```

### Check installed BIND utilities

Use this command to list installed `bind-utils` packages:

```bash
rpm -qa | grep bind-utils
```

### Install BIND utilities

To install BIND utilities using `yum`:

```bash
yum install bind-utils
```

### List files installed by bind-utils

To list all files installed by the `bind-utils` package:

```bash
rpm -ql bind-utils
```

### List configuration files for bind-utils

To list configuration files for the `bind-utils` package:

```bash
rpm -qc bind-utils
```

### List documentation files for bind-utils

To list documentation files for the `bind-utils` package:

```bash
rpm -qd bind-utils
```

### Common DNS client tools installed with bind-utils

These are the commonly used DNS client tools included with `bind-utils`:

| Path | Description |
| --- | --- |
| `/usr/bin/delv` | DNS lookup and validation utility |
| `/usr/bin/dig` | Query DNS servers for information about domains |
| `/usr/bin/host` | Simple utility for performing DNS lookups |
| `/usr/bin/mdig` | Multiple query version of `dig` |
| `/usr/bin/nslookup` | Legacy tool to query DNS servers |
| `/usr/bin/nsupdate` | Dynamic DNS update utility |

## DNS Records

DNS records define how domain names are mapped to IP addresses and other resources.

| Type | Maps | Example |
| --- | --- | --- |
| `A` | Hostname → IPv4 | `example.com → 192.168.1.1` |
| `AAAA` | Hostname → IPv6 | `example.com → 2001:0db8::ff00:0042:8329` |
| `CNAME` | Alias → canonical name | `wp.armourinfosec.com → armourinfosec.wordpress.com` |
| `NS` | Zone → authoritative nameserver | `example.com → ns1.example.com` |
| `PTR` | IP → hostname (reverse) | `192.168.1.1 → example.com` |
| `SOA` | Zone authority metadata | primary NS + serial/timers |
| `HINFO` | Host hardware/OS info | `"Intel i7" "Linux"` |
| `MX` | Domain → mail server | `mail.example.com (priority 10)` |
| `TXT` | Arbitrary text (SPF/DKIM) | `"v=spf1 …"` |

### A (Address Mapping) Records

- Specifies an IPv4 address for a given host.
- Example:
    `example.com → 192.168.1.1`

### AAAA (IPv6 Address) Records

- Specifies an IPv6 address for a given host.
- Example:
    `example.com → 2001:0db8::ff00:0042:8329`

### CNAME (Canonical Name) Records

- Specifies an alias for another domain name.
- Example:
    `wp.armourinfosec.com → armourinfosec.wordpress.com → 20.20.20.20`

### NS (Name Server) Records

- Specifies an authoritative name server for a given domain.
- Example:
    `example.com → ns1.example.com`

### PTR (Pointer) Records

- Used for reverse DNS lookups (mapping IP addresses to hostnames).
- Example:
    `192.168.1.1 → example.com`

### SOA (Start of Authority) Records

- Specifies authoritative information about a DNS zone, including the primary nameserver and zone serial number.
- Example:

    ```text
    example.com. IN SOA ns1.example.com. admin.example.com. (
        2025031101 ; serial
        3600       ; refresh
        1800       ; retry
        1209600    ; expire
        86400      ; minimum TTL
    )
    ```

### HINFO (Host Information) Records

- Provides information about a host's hardware and operating system.
- Example:
    `example.com. IN HINFO "Intel i7" "Linux"`

> [!WARNING]
> `HINFO` records disclose hardware and OS details to anyone who queries them, aiding attacker fingerprinting. Avoid publishing them on Internet-facing zones.

### MX (Mail Exchanger) Records

- Specifies a mail exchange server for a domain.
- Example:
    `example.com → mail.example.com (priority 10)`

### TXT (Text) Records

- Holds arbitrary text data for a domain.
- Often used for SPF (Sender Policy Framework) or DKIM (DomainKeys Identified Mail) records.
- Example:

```text
    example.com. IN TXT "v=spf1 include:_spf.google.com ~all"
```

## Examples

Query an `A` record with `dig` against a specific resolver:

```bash
dig example.com @8.8.8.8
```

Perform a quick lookup with `host`:

```bash
host example.com
```

Query an `MX` record:

```bash
dig MX example.com
```

## Best Practices

- Prefer `dig` for troubleshooting — it exposes flags, sections, and timing that `nslookup` hides.
- Use `delv` (or `dig +dnssec`) when you need to confirm DNSSEC validation, not just resolution.
- Query a specific server with `@server` to bypass local caching when diagnosing authoritative data.

## References

- `dig(1)`, `host(1)`, `nslookup(1)`, `delv(1)` — `bind-utils` manual pages.
- RFC 1035 — resource record types (`A`, `NS`, `CNAME`, `SOA`, `MX`, `TXT`, `PTR`).
- RFC 3596 — DNS Extensions to Support IPv6 (`AAAA` records).
- RFC 7208 — Sender Policy Framework (SPF) records published as `TXT`.

## Related

- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub.
- [dig](dig.md) — detailed `dig` usage and flags.
- [host](host.md) — the `host` lookup tool.
- [nslookup](nslookup.md) — legacy `nslookup` query tool.
- [DNS-Server-Types](DNS-Server-Types.md) — the servers these tools query.
- [Forward-Zone](Forward-Zone.md) — the records defined here in a live zone file.
