# Network Operating Systems and Security Solutions

## Overview

Beyond host-level firewalls, network security is delivered by purpose-built **operating systems** and **detection/prevention platforms** that run on routers, dedicated appliances, or virtual machines. This note surveys the major categories — router operating systems, firewall operating systems, and intrusion detection/prevention systems — and where each fits in a defence-in-depth architecture.

> [!NOTE]
> **Layered defence**
> A host firewall (iptables/firewalld/UFW) protects a single machine. A firewall OS or router OS protects a **network segment** at the perimeter, and IDS/IPS adds **visibility and active blocking** on top of both.

## Architecture

```mermaid
flowchart LR
    Internet --> RTR[Router OS<br/>OpenWrt / VyOS]
    RTR --> FW[Firewall OS<br/>pfSense]
    FW --> IDS[IDS / IPS<br/>Snort / Suricata]
    IDS --> LAN[Internal LAN]
```

## Router Operating Systems (Router OS)

| Platform | Base | Description |
| :-- | :-- | :-- |
| **DD-WRT** | Linux | Popular open-source firmware for wireless routers, offering enhanced features, better performance, and advanced network management compared to stock firmware. |
| **OpenWrt** | Linux | Highly customizable open-source OS for embedded devices, widely used on routers for robust networking, security, and package management. |
| **VyOS** | Linux | Open-source network OS designed for advanced routing, firewalling, VPN, and network virtualization on standard x86 hardware. |
| **IPCop** | Linux | Dedicated firewall and router distribution designed for easy setup and management in small office / home office environments. |

## Firewall Operating System (Firewall OS)

| Platform | Base | Description |
| :-- | :-- | :-- |
| **pfSense** | FreeBSD | Open-source firewall and router platform offering powerful firewalling, VPN, routing, DHCP, and IDS/IPS capabilities for enterprise and home use. |

## Intrusion Detection & Prevention

### Intrusion Detection System (IDS)

- **Purpose:** Detect malicious or abnormal activity on the network.
- Works by monitoring network traffic and comparing it against signatures or behavioural rules.
- **Does NOT block traffic automatically** (alert-only).

**Examples:** Snort, Suricata, Zeek (Bro).

### Intrusion Prevention System (IPS)

- **Purpose:** Actively blocks or prevents malicious traffic.
- Can drop suspicious packets in real time.
- Often built into modern **Next-Gen Firewalls (NGFW)**.

**Examples:** Snort (in IPS mode), Suricata, Cisco Firepower.

> [!TIP]
> **IDS vs. IPS placement**
> An **IDS** typically sits out-of-band on a mirror/SPAN port — it observes and alerts. An **IPS** sits inline in the traffic path so it can drop packets, which means it must be sized for throughput and fail-open/closed behaviour must be planned.

## Quick Comparison

| Category | Purpose | Examples |
| --- | --- | --- |
| Router OS | Routing, NAT, VPN, sometimes firewall | DD-WRT, OpenWrt, VyOS, IPCop |
| Firewall OS | Packet filtering, stateful firewall, VPN, IDS/IPS integration | pfSense |
| IDS | Detect suspicious/malicious activity (alert only) | Snort, Suricata, Zeek |
| IPS | Detect + block malicious traffic in real time | Snort (IPS mode), Suricata, Cisco Firepower |

## Security Considerations

- **IDS is not prevention.** Alert-only systems require an operator or SOAR pipeline to act on findings; alerts without response provide little protection.
- **Signature freshness matters.** Both IDS and IPS depend on current rule sets — stale signatures miss new attack patterns.
- **Inline IPS is a single point of failure.** Plan for fail-open vs. fail-closed and for capacity, or an IPS outage can take the network down with it.
- **Segment before you inspect.** Router and firewall operating systems let you separate zones (DMZ, LAN, guest) so an IDS/IPS has meaningful boundaries to enforce.

## References

- pfSense documentation — https://docs.netgate.com/pfsense/
- OpenWrt documentation — https://openwrt.org/docs/start
- Snort — https://www.snort.org/
- Suricata — https://suricata.io/
- Zeek — https://zeek.org/

## Related

- [Firewall-Network-Security-Barrier](Firewall-Network-Security-Barrier.md) — firewall concepts overview
- [IPTables-Configuration-and-Management](IPTables-Configuration-and-Management.md) — netfilter rule management
- [Firewalld](Firewalld.md) — dynamic firewall daemon
- [Uncomplicated-Firewall(ufw)](Uncomplicated-Firewall(ufw).md) — simplified firewall front-end
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
