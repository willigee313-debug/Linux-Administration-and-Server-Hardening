# Firewalld

## Overview

**Firewalld** is a firewall management tool for Linux that provides a dynamic way to configure and manage firewall rules. It is a modern replacement for **iptables** and is commonly used in **Red Hat-based distributions** like RHEL, CentOS, Fedora, and Rocky Linux.

Unlike raw iptables, firewalld introduces **zones** (trust levels bound to interfaces or sources), a **runtime vs. permanent** configuration split, and a D-Bus API so rules can change on the fly without dropping active connections. This note covers status checks, installation, rule management, zones, rich rules, NAT, and troubleshooting.

## Key Features of Firewalld

| Feature | Description |
| :-- | :-- |
| **Zone-based management** | Different network interfaces can be assigned different security levels using zones. |
| **Dynamic firewall** | Rules can be changed without dropping active connections. |
| **D-Bus interface** | Allows services and applications to request firewall changes dynamically. |
| **IPv4 and IPv6 support** | Works with both protocols. |
| **Rich rules** | Provide more granular control compared to iptables. |
| **Easy-to-use commands** | Uses `firewall-cmd` for managing firewall rules. |

## Architecture

```mermaid
flowchart TB
    subgraph Zones[Firewalld zones by trust level]
      T[trusted] --> INT[internal] --> H[home] --> W[work] --> PUB[public] --> EXT[external] --> DMZ[dmz] --> BLK[block] --> DROP[drop]
    end
    IF[Network interface / source] -->|assigned to| PUB
    PUB --> RT[Runtime config]
    PUB --> PERM[Permanent config]
    RT -.--runtime-to-permanent.-> PERM
```

> [!NOTE]
> **Runtime vs. permanent**
> Commands **without** `--permanent` change only the **runtime** configuration (lost on reload/restart). Commands **with** `--permanent` change the **on-disk** configuration but do not take effect until `firewall-cmd --reload`. Use `--runtime-to-permanent` to promote tested runtime rules to permanent.

## Checking Firewalld Status

- Check if firewalld is installed:

```bash
rpm -qa | grep firewalld
```

- Display detailed package information about firewalld:

```bash
rpm -qi firewalld
```

- List all files installed by the firewalld package:

```bash
rpm -ql firewalld
```

- List all configuration files provided by the firewalld package:

```bash
rpm -qc firewalld
```

- List all documentation files included in the firewalld package:

```bash
rpm -qd firewalld
```

- Check the current status of firewalld (running or stopped):

```bash
systemctl status firewalld
```

- Stop and disable iptables to ensure firewalld takes control:

```bash
systemctl disable iptables
```

```bash
systemctl stop iptables
```

- Enable firewalld to start on boot and immediately start the service:

```bash
systemctl enable firewalld
```

```bash
systemctl start firewalld
```

- Verify the firewalld status again after enabling and starting it:

```bash
systemctl status firewalld
```

## Installing and Managing Firewalld

- Install firewalld if not already installed:

```bash
yum install firewalld -y
```

- Restart firewalld after making configuration changes:

```bash
systemctl restart firewalld
```

- Reload firewalld without restarting the service to apply changes:

```bash
firewall-cmd --reload
```

## Listing Firewalld Rules

- Show all active zones and their associated services:

```bash
firewall-cmd --get-active-zones
```

- Display all active firewall rules for the default zone:

```bash
firewall-cmd --list-all
```

## Adding Firewall Rules

### Allow Incoming Traffic on Specific Ports

- Allow FTP traffic on port 21:

```bash
firewall-cmd --permanent --add-port=21/tcp
```

- Allow HTTP traffic on port 80:

```bash
firewall-cmd --permanent --add-port=80/tcp
```

- Allow HTTPS traffic on port 443:

```bash
firewall-cmd --permanent --add-port=443/tcp
```

- Allow SSH traffic on port 22:

```bash
firewall-cmd --permanent --add-port=22/tcp
```

- Allow DNS traffic on UDP port 53:

```bash
firewall-cmd --permanent --add-port=53/udp
```

- Allow DHCP traffic on UDP port 67:

```bash
firewall-cmd --permanent --add-port=67/udp
```

- Reload firewalld without restarting the service to apply changes:

```bash
firewall-cmd --reload
```

### Blocking Specific IPs or Subnets

- Drop all incoming traffic from a specific IP address:

```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.61" drop'
```

- Drop all incoming traffic from an entire subnet:

```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="20.20.20.0/24" drop'
```

### Blocking Outgoing Traffic

- Block outbound traffic to Google DNS (8.8.8.8):

```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" destination address="8.8.8.8" reject'
```

- Block outbound HTTPS traffic to Google's IP range:

```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" destination address="172.217.197.0/24" port protocol="tcp" port="443" reject'
```

## Deleting Firewall Rules

- Remove an Allowed Port

```bash
firewall-cmd --permanent --remove-port=80/tcp
```

- Remove an Allowed Service

```bash
firewall-cmd --permanent --remove-service=https
```

- Remove a Rich Rule

```bash
firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.1.61" drop'
```

```bash
firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="20.20.20.0/24" drop'
```

## Changing Zones in Firewalld

- Change the Default Zone

```bash
firewall-cmd --set-default-zone=public
```

- Assign a Network Interface to a Zone

```bash
firewall-cmd --zone=trusted --change-interface=enp0s8 --permanent
```

```bash
firewall-cmd --reload
```

## Reloading and Saving Rules

- Reload Firewalld to apply changes:

```bash
firewall-cmd --reload
```

- Save runtime rules to permanent:

```bash
firewall-cmd --runtime-to-permanent
```

## Restoring Firewalld Rules

- To reset Firewalld to its factory defaults:

```bash
firewall-cmd --complete-reload
```

- Or when firewalld is not running:

```bash
firewall-offline-cmd --clear
```

## Allowing and Denying Services

Firewalld ships with predefined **service** definitions (name + ports) so you can allow an application without memorizing its ports.

- Allow NFS service (instead of specifying multiple ports):

```bash
firewall-cmd --permanent --add-service=nfs
```

- Deny (remove) Samba service:

```bash
firewall-cmd --permanent --remove-service=samba
```

## Port Ranges

- Allow port range 3000–3010 for TCP:

```bash
firewall-cmd --permanent --add-port=3000-3010/tcp
```

## Forwarding Rules

- Forward all traffic from port 8080 to port 80:

```bash
firewall-cmd --permanent --add-forward-port=port=8080:proto=tcp:toport=80
```

- Forward port 2222 on local system to SSH (22) on another host:

```bash
firewall-cmd --permanent --add-forward-port=port=2222:proto=tcp:toaddr=192.168.1.100:toport=22
```

## Masquerading (NAT)

- Enable masquerading in a zone:

```bash
firewall-cmd --zone=public --add-masquerade --permanent
```

- Disable masquerading:

```bash
firewall-cmd --zone=public --remove-masquerade --permanent
```

> [!TIP]
> **Masquerading enables source NAT**
> Enable masquerade on the **outbound** zone so internal hosts can share a single public IP. It is required for the forward-port-to-another-host examples above to work across networks.

## Rich Rules – Logging & Rate Limiting

- Log packets from a specific source:

```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.50" log prefix="FW-DROP " level="info" drop'
```

- Limit SSH login attempts (max 3 connections per minute):

```bash
firewall-cmd --permanent --add-rich-rule='rule service name="ssh" limit value="3/m" accept'
```

## ICMP Rules

- Block all ping requests:

```bash
firewall-cmd --permanent --add-icmp-block=echo-request
```

- Allow ping again:

```bash
firewall-cmd --permanent --remove-icmp-block=echo-request
```

- List available ICMP types:

```bash
firewall-cmd --get-icmptypes
```

## Interface & Zone Management

- Add interface to internal zone:

```bash
firewall-cmd --zone=internal --add-interface=enp0s3 --permanent
```

- List interfaces per zone:

```bash
firewall-cmd --get-active-zones
```

## IPv6 Rules

- Block all incoming traffic from IPv6 subnet:

```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv6" source address="2001:db8::/32" drop'
```

## Temporary Rules (Runtime Only)

- Allow HTTP for this session only:

```bash
firewall-cmd --add-service=http
```

- Remove it again:

```bash
firewall-cmd --remove-service=http
```

## Zone Management (Granular Control)

- List all available zones:

```bash
firewall-cmd --get-zones
```

- Show details of a specific zone:

```bash
firewall-cmd --zone=public --list-all
```

- Create a custom zone:

```bash
firewall-cmd --permanent --new-zone=dmz-test
```

- Delete a custom zone:

```bash
firewall-cmd --permanent --delete-zone=dmz-test
```

## Source-Based Rules

- Allow traffic only from a specific subnet:

```bash
firewall-cmd --permanent --zone=public --add-source=192.168.10.0/24
```

- Remove source from zone:

```bash
firewall-cmd --permanent --zone=public --remove-source=192.168.10.0/24
```

## Source Port Rules

- Allow packets coming **from source port 12345** (uncommon case):

```bash
firewall-cmd --permanent --add-source-port=12345/tcp
```

- Allow a range of source ports:

```bash
firewall-cmd --permanent --add-source-port=10000-10100/udp
```

## Helpers (Protocols Requiring Tracking)

- List all helpers:

```bash
firewall-cmd --get-helpers
```

- Add FTP helper (for passive FTP):

```bash
firewall-cmd --permanent --add-helper=ftp
```

## Complex Rich Rules

- Allow SSH only from subnet, drop others:

```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept'
firewall-cmd --permanent --add-rich-rule='rule service name="ssh" drop'
```

- Allow HTTP/HTTPS but log before accepting:

```bash
firewall-cmd --permanent --add-rich-rule='rule service name="http" log prefix="HTTP-ACCEPT " level="info" accept'
firewall-cmd --permanent --add-rich-rule='rule service name="https" log prefix="HTTPS-ACCEPT " level="info" accept'
```

- Limit ICMP echo requests (max 5 per second):

```bash
firewall-cmd --permanent --add-rich-rule='rule protocol value="icmp" limit value="5/s" accept'
```

## Lockdown Mode (Restrictive Control)

- Enable lockdown (only allow services in lockdown whitelist):

```bash
firewall-cmd --lockdown-on
```

- Disable lockdown:

```bash
firewall-cmd --lockdown-off
```

- Allow a service in lockdown:

```bash
firewall-cmd --permanent --add-lockdown-whitelist-service=ssh
```

- Whitelist a user for lockdown:

```bash
firewall-cmd --permanent --add-lockdown-whitelist-user=root
```

## Rich Rule with Timeouts (Ephemeral Rules)

- Allow port 9090 for 60 seconds only:

```bash
firewall-cmd --add-port=9090/tcp --timeout=60
```

> [!TIP]
> **Timeouts prevent lockouts**
> A `--timeout` rule is perfect for testing risky changes remotely: if the new rule locks you out, it auto-expires and access is restored.

## Debugging & Monitoring

- List runtime configuration:

```bash
firewall-cmd --list-all --zone=public
```

- Show all runtime rules in detail:

```bash
firewall-cmd --list-all-zones
```

- Watch logs in real-time (example with journalctl):

```bash
journalctl -xe -u firewalld
```

- Test effective rules in nftables backend:

```bash
nft list ruleset
```

## Advanced IPv6 Examples

- Drop all IPv6 traffic except SSH:

```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv6" service name="ssh" accept'
firewall-cmd --permanent --add-rich-rule='rule family="ipv6" drop'
```

- Block multicast (ff00::/8):

```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv6" source address="ff00::/8" drop'
```

## Best Practices

- **Always pair `--permanent` with `--reload`.** Permanent changes are inert until reloaded; test in runtime first, then promote with `--runtime-to-permanent`.
- **Right-size the default zone.** `public` is a sensible internet-facing default; use `internal`/`trusted` only for genuinely trusted segments.
- **Prefer services over raw ports** where a predefined service exists — it is self-documenting and stays correct if the app's ports change.
- **Use `--timeout` for remote changes** so a mistake self-heals instead of locking you out.
- **Log denies** with rich-rule `log prefix=...` so blocked traffic is visible for auditing.

## Security Considerations

- A misassigned interface (e.g., an internet-facing NIC placed in `trusted`) effectively disables the firewall for that link — verify zone/interface bindings with `--get-active-zones`.
- Rich-rule ordering and zone precedence can produce surprising results; validate the effective ruleset with `firewall-cmd --list-all` and, if needed, `nft list ruleset`.
- Enable **lockdown mode** on hardened hosts to prevent local applications from silently opening the firewall via the D-Bus API.

## Troubleshooting

| Symptom | Likely cause / fix |
| :-- | :-- |
| Rule added but has no effect | You used `--permanent` without `firewall-cmd --reload`. |
| Rules vanish after reload | You added a runtime rule (no `--permanent`); promote with `--runtime-to-permanent`. |
| Traffic allowed unexpectedly | Interface is in a more-trusted zone than intended — check `--get-active-zones`. |
| Need live event visibility | `journalctl -xe -u firewalld` and rich-rule logging. |

## References

- firewalld documentation — https://firewalld.org/documentation/
- Red Hat: Using and configuring firewalld — https://access.redhat.com/documentation/

## Related

- [Firewall-Network-Security-Barrier](Firewall-Network-Security-Barrier.md) — firewall concepts overview
- [IPTables-Configuration-and-Management](IPTables-Configuration-and-Management.md) — underlying netfilter layer
- [Uncomplicated-Firewall(ufw)](Uncomplicated-Firewall(ufw).md) — alternative simplified front-end
- [Network-Operating-Systems-and-Security-Solutions](Network-Operating-Systems-and-Security-Solutions.md) — network security context
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
