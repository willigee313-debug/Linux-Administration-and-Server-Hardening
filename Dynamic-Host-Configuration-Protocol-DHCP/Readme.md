# Dynamic Host Configuration Protocol (DHCP)

ISC DHCP server: pools, reservations, exclusions, and client allow/deny policy.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Automated IP address assignment with the ISC DHCP server. This module covers address pools, static reservations, exclusion ranges, and controlling which clients receive leases through known/unknown handling and allow/deny lists — the counterpart to DNS in a managed network and a prerequisite for PXE boot.

## Learning Objectives

By the end of this module you will be able to:

- Configure address pools, reservations, and exclusion ranges
- Control leasing with known/unknown client handling and allow/deny lists
- Integrate DHCP options for gateway, DNS, and PXE next-server

## Topics Covered

This module contains **6 notes**.

| Note | Topic |
| --- | --- |
| [Allow-for-Known-and-Unknown](Allow-for-Known-and-Unknown.md) | Allow for Known and Unknown |
| [Black-or-Deny-list-Clients](Black-or-Deny-list-Clients.md) | Black or Deny list Clients |
| [Configure-DHCP-Exclusion-Range](Configure-DHCP-Exclusion-Range.md) | Configure DHCP Exclusion Range |
| [DHCP-Server](DHCP-Server.md) | DHCP Server |
| [Reserve-IP-Address](Reserve-IP-Address.md) | Reserve IP Address |
| [White-or-Allow-list-Clients](White-or-Allow-list-Clients.md) | White or Allow list Clients |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Reserve addresses for infrastructure and servers; pool the rest
- Document the authoritative DHCP server per segment to avoid rogue servers
- Keep lease times appropriate to the network's churn

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Deny unknown clients on sensitive segments and log lease assignments
- Guard against rogue DHCP with switch-level DHCP snooping
- Restrict who can edit the dhcpd configuration and reload the service

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Clients not getting addresses | Check the service is authoritative, the subnet declaration matches the interface, and relay/helper settings |
| Reservation ignored | Verify the client MAC and that the host is not also matched by a dynamic pool |

## References

- [ISC DHCP / Kea documentation](https://www.isc.org/dhcp/)
- [dhcpd.conf(5) man page](https://man.archlinux.org/man/dhcpd.conf.5)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Domain Name System (DNS)](../Domain-Name-System-DNS/Readme.md) — related module
- [Network Configuration](../Network-Configuration/Readme.md) — related module
- [TFTP and PXE Boot Server](../TFTP-and-PXE-Boot-Server/Readme.md) — related module
