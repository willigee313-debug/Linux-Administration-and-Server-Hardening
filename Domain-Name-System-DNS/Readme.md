# Domain Name System (DNS)

Authoritative and caching BIND servers, zones, and resolution troubleshooting.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Name resolution infrastructure with BIND. This module covers server types (recursive/caching, master/authoritative, slave, forwarders), forward and reverse zones, multiple-zone setups, and the client tools (dig, host, nslookup) used to verify and troubleshoot resolution.

## Learning Objectives

By the end of this module you will be able to:

- Distinguish authoritative, recursive/caching, forwarding, and slave roles
- Author forward and reverse zones and configure zone transfers
- Verify and troubleshoot resolution with dig, host, and nslookup

## Topics Covered

This module contains **12 notes**.

| Note | Topic |
| --- | --- |
| [Caching-Nameserver](Caching-Nameserver.md) | Caching Nameserver |
| [DNS-Server-Types](DNS-Server-Types.md) | DNS Server Types |
| [Dns-Client-Tools](Dns-Client-Tools.md) | Dns Client Tools |
| [Forward-Zone](Forward-Zone.md) | Forward Zone |
| [Forwarders-Nameserver](Forwarders-Nameserver.md) | Forwarders Nameserver |
| [Master-Nameserver](Master-Nameserver.md) | Master Nameserver |
| [Multiple-Zone-Configuration](Multiple-Zone-Configuration.md) | Multiple Zone Configuration |
| [Reverse-Zone](Reverse-Zone.md) | Reverse Zone |
| [Slave-DNS-Server](Slave-DNS-Server.md) | Slave DNS Server |
| [dig](dig.md) | dig |
| [host](host.md) | host |
| [nslookup](nslookup.md) | nslookup |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Keep serial numbers disciplined (increment on every zone change)
- Separate authoritative and recursive roles on internet-facing deployments
- Set sensible TTLs balancing change agility and cache efficiency

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Restrict recursion and zone transfers to trusted clients to prevent abuse and data leakage
- Consider DNSSEC for zone integrity and response validation
- Keep BIND patched — it is a frequent target

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Slave zone will not transfer | Check `allow-transfer`, the master's `also-notify`, and serial numbers |
| Resolution intermittently fails | Verify forwarders, recursion settings, and firewall rules on port 53 TCP/UDP |

## References

- [BIND 9 / ISC documentation](https://bind9.readthedocs.io/)
- [RFC 1034/1035](https://www.rfc-editor.org/rfc/rfc1035)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [DNS & DHCP Infrastructure (Project 03)](../Enterprise-Projects/Project-03-DNS-and-DHCP-Infrastructure.md) — capstone build
- [Lab 09 — DNS Server](../Practical-Labs/Lab-09-DNS-Server.md) — hands-on for this module
- [Network Configuration](../Network-Configuration/Readme.md) — related module
- [Dynamic Host Configuration Protocol (DHCP)](../Dynamic-Host-Configuration-Protocol-DHCP/Readme.md) — related module
- [TFTP and PXE Boot Server](../TFTP-and-PXE-Boot-Server/Readme.md) — related module
