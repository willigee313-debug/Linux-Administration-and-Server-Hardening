# Enterprise Projects

## Overview

The **Enterprise Projects** are the capstone tier of this course. Every module and lab elsewhere in the vault teaches one service in isolation — Apache on its own, BIND on its own, LDAP on its own. In a real environment nothing runs alone: a web tier depends on DNS, DNS depends on a hardened baseline OS, that baseline depends on central authentication, and all of it needs to be visible to a monitoring stack and reproducible by automation. These ten notes take the individual [module](../Readme.md) and [lab](../Practical-Labs/Readme.md) content and re-assemble it into production-shaped builds — each one scoped like a Red Hat reference architecture or a CIS-hardened build guide, with a business scenario, an architecture diagram, real configuration, a security-control mapping, ordered deployment steps, and validation checks.

They are meant to be read (and built) **after** the corresponding module notes, not instead of them. Where a project note says "see [Apache Web Server](../Apache-Web-Server/Readme.md) for the underlying mechanics," that module is the reference and the project is the integration.

> [!NOTE]
> **How to use this index**
> Each project stands alone technically, but several reuse hosts and services from earlier ones (see [Suggested Order](#suggested-order) and [Shared Lab Estate](#shared-lab-estate)). Build them in sequence on a shared lab network for the closest approximation of a real enterprise estate, or cherry-pick a single project if you only need that integration pattern.

## Project Index

| Project | Scenario | Services Integrated |
|---|---|---|
| [Project-01-Enterprise-Linux-Server](Project-01-Enterprise-Linux-Server.md) — Project 01 — Enterprise Linux Server Baseline | Golden-image hardened Linux host every other build clones from | Users/Groups/sudo, SSH, firewalld, SELinux/AppArmor, auditd, unattended patching |
| [Project-02-Secure-Web-Hosting-Platform](Project-02-Secure-Web-Hosting-Platform.md) — Project 02 — Secure Web Hosting Platform | Multi-tenant web hosting provider, three vhosts on one box | Apache name-based vhosts, Let's Encrypt TLS, PHP-FPM pools, ModSecurity/OWASP CRS, fail2ban |
| [Project-03-DNS-and-DHCP-Infrastructure](Project-03-DNS-and-DHCP-Infrastructure.md) — Project 03 — DNS & DHCP Infrastructure | Multi-site enterprise needing redundant split-horizon DNS + DDNS | BIND9 primary/secondary, TSIG zone transfer, ISC DHCP, split-horizon views |
| [Project-04-Secure-File-Server](Project-04-Secure-File-Server.md) — Project 04 — Secure File Server | Mixed Windows/Linux client office needing one identity, one ACL model | LDAP, Samba (SMB/CIFS), NFSv4 + Kerberos, POSIX ACLs, XFS quotas, LVM snapshots |
| [Project-05-Central-Authentication](Project-05-Central-Authentication.md) — Project 05 — Central Authentication | 60-host estate replacing local accounts ahead of a SOC 2 audit | OpenLDAP, LDAPS, SSSD, `sudoRole` centralized sudo, ppolicy |
| [Project-06-PXE-Deployment-Environment](Project-06-PXE-Deployment-Environment.md) — Project 06 — PXE Deployment Environment | Bare-metal hosting provider re-imaging 40+ servers/quarter | DHCP network bootstrap, TFTP, Kickstart/Preseed automated installs |
| [Project-07-Enterprise-Firewall](Project-07-Enterprise-Firewall.md) — Project 07 — Enterprise Firewall & Segmentation | Flat network failing PCI-DSS segmentation review | nftables perimeter/router, firewalld host zones, DNAT, Suricata IDS/IPS, SIEM log shipping |
| [Project-08-OpenWrt-Branch-Office](Project-08-OpenWrt-Branch-Office.md) — Project 08 — OpenWrt Branch Office | 12-user branch office replacing a consumer router | OpenWrt UCI, VLAN segmentation, guest Wi-Fi isolation, WireGuard site-to-site, QoS |
| [Project-09-Monitoring-Server](Project-09-Monitoring-Server.md) — Project 09 — Central Monitoring Server | 40-host estate with no centralized alerting or log visibility | Prometheus, Node/Blackbox Exporter, Grafana, Alertmanager, Loki/rsyslog |
| [Project-10-Infrastructure-Automation](Project-10-Infrastructure-Automation.md) — Project 10 — Infrastructure Automation | Estate-wide config drift flagged in an audit finding | Ansible control node, CIS hardening role, web/DNS/monitoring roles, idempotent playbooks |

## Suggested Order

1. **[Project-01-Enterprise-Linux-Server](Project-01-Enterprise-Linux-Server.md)** first, always — it is the golden image every other project assumes as a starting point (least-privilege accounts, SSH hardening, firewalld default-deny, MAC, auditd).
2. **[Project-05-Central-Authentication](Project-05-Central-Authentication.md)** early — once more than one host exists, local accounts become the thing every later project has to work around. Standing up LDAP/SSSD before the file, web, or monitoring tiers avoids re-plumbing authentication later.
3. **[Project-03-DNS-and-DHCP-Infrastructure](Project-03-DNS-and-DHCP-Infrastructure.md)** before anything that needs stable name resolution or DDNS-registered leases — the web, file, and monitoring projects all resolve internal names against it.
4. **[Project-02-Secure-Web-Hosting-Platform](Project-02-Secure-Web-Hosting-Platform.md)** and **[Project-04-Secure-File-Server](Project-04-Secure-File-Server.md)** — the two "consumer" service tiers; either order is fine, both build directly on Projects 01, 03, and 05.
5. **[Project-06-PXE-Deployment-Environment](Project-06-PXE-Deployment-Environment.md)** once you have a DHCP server to extend (Project 03) and a golden-image baseline (Project 01) worth deploying at scale via Kickstart/Preseed.
6. **[Project-07-Enterprise-Firewall](Project-07-Enterprise-Firewall.md)** after real services exist behind it — segmentation and DNAT rules are meaningless without a DMZ workload (Project 02) and a LAN workload (Project 04/05) to protect.
7. **[Project-09-Monitoring-Server](Project-09-Monitoring-Server.md)** last among the "build" projects — it onboards Projects 01–07 as monitored targets, so standing it up earlier just means re-registering targets as they come online.
8. **[Project-08-OpenWrt-Branch-Office](Project-08-OpenWrt-Branch-Office.md)** is self-contained (a separate branch-office appliance) and can be built any time after Project 07, since it reuses the same firewall-zone and WireGuard mental model on OpenWrt's UCI system.
9. **[Project-10-Infrastructure-Automation](Project-10-Infrastructure-Automation.md)** last — it codifies Projects 01, 02, and 03 as Ansible roles, which is far more useful once you've felt the pain of configuring them by hand at least once.

## Shared Lab Estate

Baseline hosts referenced across multiple projects. Individual project notes define additional service-specific hosts; this table is the common substrate.

| Hostname | Role | Suggested IP | OS | vCPU / RAM / Disk |
|---|---|---|---|---|
| `baseline-01` | Golden-image template (Project 01) | 10.10.10.10/24 | Rocky Linux 9 / Debian 12 | 2 / 2 GB / 20 GB |
| `ldap-01` | Central authentication (Project 05) | 10.10.10.20/24 | Rocky Linux 9 | 2 / 2 GB / 20 GB |
| `dns-01` / `dns-02` | BIND9 primary / secondary (Project 03) | 10.10.10.30/24, 10.10.10.31/24 | Debian 12 | 1 / 1 GB / 10 GB |
| `dhcp-01` | ISC DHCP + PXE boot server (Projects 03, 06) | 10.10.10.32/24 | Debian 12 | 1 / 1 GB / 10 GB |
| `web-01` | Apache multi-vhost host (Project 02) | 10.10.20.10/24 (DMZ) | Ubuntu 22.04 | 2 / 2 GB / 20 GB |
| `file-01` | Samba + NFS file server (Project 04) | 10.10.10.40/24 | Rocky Linux 9 | 2 / 4 GB / 60 GB |
| `fw-01` | nftables perimeter router (Project 07) | WAN + 10.10.10.1/24 + 10.10.20.1/24 | Debian 12 | 2 / 2 GB / 20 GB |
| `mon-01` | Prometheus/Grafana/Loki (Project 09) | 10.10.10.50/24 | Debian 12 | 2 / 4 GB / 40 GB |
| `ansible-ctl` | Ansible control node (Project 10) | 10.10.10.60/24 | Rocky Linux 9 | 1 / 1 GB / 10 GB |
| `branch-gw` | OpenWrt branch appliance (Project 08) | separate site, WAN + LAN | OpenWrt 23.05 | any x86/ARM box, 512 MB+ RAM |

> [!NOTE]
> **📸 Screenshot**
> _Capture: a diagram or virt-manager/VirtualBox host list showing the shared lab estate powered on together, to accompany this index when the full multi-project build is running._

## References

- CIS Benchmarks — Red Hat Enterprise Linux / Debian / Apache HTTP Server / BIND — [cisecurity.org](https://www.cisecurity.org/cis-benchmarks)
- NIST SP 800-53 Rev. 5 — Security and Privacy Controls for Information Systems and Organizations
- Red Hat Reference Architectures — [access.redhat.com/architect](https://access.redhat.com/architect)
- Individual project notes (below) — each carries its own service-specific reference list

## Related Notes

- [Project-01-Enterprise-Linux-Server](Project-01-Enterprise-Linux-Server.md)
- [Project-02-Secure-Web-Hosting-Platform](Project-02-Secure-Web-Hosting-Platform.md)
- [Project-03-DNS-and-DHCP-Infrastructure](Project-03-DNS-and-DHCP-Infrastructure.md)
- [Project-04-Secure-File-Server](Project-04-Secure-File-Server.md)
- [Project-05-Central-Authentication](Project-05-Central-Authentication.md)
- [Project-06-PXE-Deployment-Environment](Project-06-PXE-Deployment-Environment.md)
- [Project-07-Enterprise-Firewall](Project-07-Enterprise-Firewall.md)
- [Project-08-OpenWrt-Branch-Office](Project-08-OpenWrt-Branch-Office.md)
- [Project-09-Monitoring-Server](Project-09-Monitoring-Server.md)
- [Project-10-Infrastructure-Automation](Project-10-Infrastructure-Automation.md)
- [Practical Labs](../Practical-Labs/Readme.md) — hands-on labs each project builds on
- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
