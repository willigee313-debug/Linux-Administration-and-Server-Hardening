# Network Monitoring netstat and ss Commands

Inspecting live network state — active connections, listening sockets, and the processes bound to them — using the classic `netstat` tool and its modern replacement `ss`.

## Overview

`netstat` and `ss` answer the everyday operational questions: *What is listening on this box? Who is connected to it? Which process owns port 443?* Both read socket state from the kernel, but they differ in age and efficiency:

- **`netstat`** is the older utility, part of the `net-tools` package, and is often **not installed by default** on modern distributions.
- **`ss`** (socket statistics) is its modern replacement — faster, more capable, and **pre-installed** on most current Linux systems because it reads socket data directly from the kernel via Netlink rather than parsing `/proc`.

Learn the `ss` flags first; they are largely a superset of `netstat`'s, and the two share the same mnemonic letters (`t`, `u`, `l`, `n`, `p`).

## Concepts

The option letters shared by both tools compose into the combinations you use daily:

| Flag | Meaning |
|------|---------|
| `-t` | TCP sockets |
| `-u` | UDP sockets |
| `-l` | Listening sockets only |
| `-n` | Numeric — skip DNS/service name resolution (faster) |
| `-p` | Show the owning process (PID/program) — needs root for others' sockets |
| `-a` | All sockets (listening + established) |
| `-s` | Summary statistics |

> [!TIP]
> The combination `ss -ltnp` ("listening TCP, numeric, with process") is the single most useful command for auditing a server's exposed surface — it lists every TCP port that is open to the network and the program behind each one.

## Installing netstat (if not available)

- RHEL / CentOS / Fedora:

```bash
yum install net-tools
```

- Debian / Ubuntu:

```bash
apt install net-tools
```

- The `ss` command is usually available out of the box on modern Linux systems.

## netstat Usage

- Display active connections

```bash
netstat
```

- Show all sockets (listening + established)

```bash
netstat -a
```

- Show numerical addresses (skip DNS lookups)

```bash
netstat -na
```

- Show only TCP connections

```bash
netstat -t
```

- Show only UDP connections

```bash
netstat -u
```

- Show both TCP and UDP connections

```bash
netstat -tu
```

- Show TCP & UDP in numeric format (faster)

```bash
netstat -tun
```

- Show listening TCP & UDP ports

```bash
netstat -ltun
```

- Show listening services with numeric IPs and process details

```bash
netstat -nltup
```

```bash
netstat -natup
```

## ss Usage (Modern Alternative)

- Display active connections

```bash
ss
```

- Show only listening sockets

```bash
ss -l
```

- Show all sockets (listening + active)

```bash
ss -a
```

- Show numerical addresses only

```bash
ss -na
```

- Show only TCP connections

```bash
ss -t
```

- Show only UDP connections

```bash
ss -u
```

- Show both TCP and UDP connections

```bash
ss -tu
```

- Show TCP & UDP in numeric format (fast)

```bash
ss -tun
```

- Show listening ports for TCP and UDP

```bash
ss -nltu
```

- Show listening services with numeric IPs and process details

```bash
ss -nltup
```

```bash
ss -natup
```

## netstat vs ss

| Feature       | `netstat`                           | `ss` (Recommended)                   |
|---------------|-------------------------------------|---------------------------------------|
| **Speed**     | Slower                              | Faster (direct kernel access)         |
| **Availability** | Requires `net-tools` package        | Pre-installed on most distributions   |
| **Protocols** | TCP, UDP, ICMP                      | TCP, UDP, RAW, UNIX, and more         |
| **Filtering** | Limited options                     | Advanced filtering capabilities       |

## Real-World Usage Examples

- Find which process is using a specific port (e.g., port 80)

```bash
ss -ltnp | grep :80
```

```bash
ss -ltnp | grep :445
```

```bash
ss -ltnp | grep :22
```

```bash
netstat -tulnp | grep :80
```

- Check all listening services and their PIDs

```bash
ss -ltnp
```

```bash
netstat -tulnp
```

- Display established connections only

```bash
ss -t state established
```

```bash
netstat -ant | grep ESTABLISHED
```

- Show summary of socket usage

```bash
ss -s
```

```bash
netstat -s
```

- Find connections to a specific IP

```bash
ss -tn dst 192.168.1.100
```

```bash
netstat -ant | grep 192.168.1.100
```

- Check which users/services are bound to ports

```bash
ss -lptn
```

```bash
netstat -lptn
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: Terminal output of ss -ltnp listing LISTEN sockets with Local Address:Port and the users:(("process",pid,fd)) column identifying each service_

## Best Practices

- Prefer `ss` over `netstat` on new systems — it is faster and does not require installing the deprecated `net-tools` package.
- Always add `-n` when scripting or triaging so a slow/failing DNS resolver does not stall the output.
- Run process-owner queries (`-p`) as root; otherwise the PID/program column is blank for sockets you do not own.
- Periodically snapshot `ss -ltnp` and diff it against a known-good baseline to catch newly-opened ports.

## Security Considerations

- Auditing listening sockets is a core hardening step: every open port is attack surface. CIS Benchmarks call for reviewing listening services and disabling those that are not required.
- An unexpected process listening on a high port, or an outbound `ESTABLISHED` connection to an unfamiliar address, can indicate a backdoor or C2 beacon — investigate the owning PID.
- Bind services to the minimum necessary interface (for example `127.0.0.1` for local-only databases) rather than `0.0.0.0`; verify the binding with `ss -ltnp`.

## Troubleshooting

| Symptom | Likely cause | Next step |
|---------|--------------|-----------|
| `netstat: command not found` | `net-tools` not installed | Use `ss`, or install `net-tools` |
| Process/PID column empty | Not running as root | Re-run with `sudo` |
| Port shows no listener but connection refused | Service bound to a different interface/address | `ss -ltnp` and check the Local Address column |
| Output very slow | Reverse DNS lookups | Add `-n` for numeric output |

## References

- `man ss`, `man netstat`
- iproute2 documentation — `ss(8)`
- CIS Benchmarks — *Minimize listening network services*

## Related
- [Network-Diagnostics-Commands](Network-Diagnostics-Commands.md) — connectivity testing tools
- [ifconfig-and-ip](ifconfig-and-ip.md) — interface and address management
- [Linux-Network-Configuration](Linux-Network-Configuration.md) — overall network setup
- [TCPDump-Command](../Security-Firewall-and-Monitoring/TCPDump-Command.md) — packet-level traffic inspection
- Network-Reconnaissance-Scanning — offensive networking hub
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
