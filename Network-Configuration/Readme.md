# Network Configuration

Interfaces, addressing, and diagnostics with ip/ifconfig, ss, netstat, and friends.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Configuring and troubleshooting Linux networking: assigning addresses and routes, inspecting interfaces with the modern `ip` suite (and legacy `ifconfig`), and diagnosing connectivity and sockets with ss, netstat, and other tools. This is the substrate every network service in the course depends on.

## Learning Objectives

By the end of this module you will be able to:

- Configure interfaces, addresses, and routes and make them persistent
- Inspect link, address, and routing state with the `ip` command
- Diagnose connectivity and open sockets with ss/netstat and standard tools

## Topics Covered

This module contains **8 notes**.

| Note | Topic |
| --- | --- |
| [ifconfig-and-ip](ifconfig-and-ip.md) | ifconfig and ip |
| [Linux-Network-Configuration](Linux-Network-Configuration.md) | Linux Network Configuration |
| [Network-Diagnostics-Commands](Network-Diagnostics-Commands.md) | Network Diagnostics Commands |
| [Network-Monitoring-netstat-and-ss-Commands](Network-Monitoring-netstat-and-ss-Commands.md) | Network Monitoring netstat and ss Commands |
| [NetworkManager-and-nmcli](NetworkManager-and-nmcli.md) | NetworkManager and nmcli |
| [Static-Routing-and-IP-Forwarding](Static-Routing-and-IP-Forwarding.md) | Static Routing and IP Forwarding |
| [Time-Synchronization-with-chrony](Time-Synchronization-with-chrony.md) | Time Synchronization with chrony |
| [VLANs-and-Network-Bonding](VLANs-and-Network-Bonding.md) | VLANs and Network Bonding |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Prefer the `ip` and `ss` tooling over deprecated `ifconfig`/`netstat`
- Document static addressing and keep it consistent with DNS/DHCP records
- Change remote networking behind a console/out-of-band path so a mistake is recoverable

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Bind services to specific interfaces/addresses rather than 0.0.0.0 where possible
- Disable unused interfaces and IPv6 only if truly not needed (prefer securing it)

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Host has an IP but cannot reach the internet | Check the default route and DNS resolver configuration |
| Change lost after reboot | Persist it in the distro's network config (NetworkManager/systemd-networkd/interfaces), not just with a live `ip` command |

## References

- [iproute2 documentation](https://man7.org/linux/man-pages/man8/ip.8.html)
- [systemd-networkd](https://www.freedesktop.org/software/systemd/man/systemd.network.html)
- [NetworkManager](https://networkmanager.dev/docs/)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Virtualization](../Virtualization/Readme.md) — virtual networking (bridges, NAT) for guests
- [Containers](../Containers/Readme.md) — container networking builds on these concepts
- [Domain Name System (DNS)](../Domain-Name-System-DNS/Readme.md) — related module
- [Dynamic Host Configuration Protocol (DHCP)](../Dynamic-Host-Configuration-Protocol-DHCP/Readme.md) — related module
- [Security, Firewall and Monitoring](../Security-Firewall-and-Monitoring/Readme.md) — related module
