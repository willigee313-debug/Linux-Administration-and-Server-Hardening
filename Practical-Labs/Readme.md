# Practical Labs

## Overview

This folder is the hands-on companion to the **Linux Administration & Server Hardening** course. Every topic module ([course hub](../Readme.md)) teaches a service or subsystem in isolation; the labs here string those pieces into complete, reproducible build exercises — install, configure, break, fix, validate — on real (or virtualized) hosts rather than single commands copy-pasted out of context.

Use this collection as a **build-your-own-lab-network curriculum**: work the labs roughly in numeric order (01 → 20), since later labs assume infrastructure — a base VM template, a working network, sometimes a DNS/DHCP pair — built in earlier ones. Each lab is self-contained enough to read standalone, but the numbering reflects a sane dependency order (install → users/permissions → storage → networking basics → hardening → services → advanced topics).

**Prerequisites**: a hypervisor capable of running 1-4 small Linux VMs concurrently (VirtualBox, KVM/virt-manager, Proxmox, or cloud instances all work), basic comfort with a terminal, and root/sudo access on your lab VMs. No prior admin experience is assumed — each lab explains the *why*, not just the *how*.

> [!WARNING]
> **Lab VMs only**
> Every lab in this folder assumes disposable, isolated lab VMs — never run destructive steps (disk repartitioning, `iptables -F`, PAM/SSH lockout tests, `authconfig`/`sssd` binds) against a production host.

## Lab Index

| Lab | Focus | Primary Module |
|---|---|---|
| [Lab-01-Linux-Installation](Lab-01-Linux-Installation.md) | Bare-metal/VM install, partitioning, first boot | [Introduction to Linux](../Introduction-to-Linux/Readme.md) |
| [Lab-02-User-Administration](Lab-02-User-Administration.md) | Users, groups, password aging, sudo | [Users, Groups & Permissions](../Users-Groups-and-Permissions/Readme.md) |
| [Lab-03-Permission-Management](Lab-03-Permission-Management.md) | chmod/chown, ACLs, SUID/SGID, sticky bit | [Users, Groups & Permissions](../Users-Groups-and-Permissions/Readme.md) |
| [Lab-04-LVM-Configuration](Lab-04-LVM-Configuration.md) | Physical/volume/logical volumes, resizing | [File System & Disk Management](../File-System-and-Disk-Management/Readme.md) |
| [Lab-05-RAID-Configuration](Lab-05-RAID-Configuration.md) | Software RAID with `mdadm`, degraded-array recovery | [File System & Disk Management](../File-System-and-Disk-Management/Readme.md) |
| [Lab-06-SSH-Hardening](Lab-06-SSH-Hardening.md) | Key auth, `sshd_config` hardening, fail2ban | [SSH Secure Shell Server](../SSH-Secure-Shell-Server/Readme.md) |
| [Lab-07-Firewall-Configuration](Lab-07-Firewall-Configuration.md) | firewalld/nftables/ufw zones and rules | [Security, Firewall & Monitoring](../Security-Firewall-and-Monitoring/Readme.md) |
| [Lab-08-Apache-Deployment](Lab-08-Apache-Deployment.md) | Virtual hosts, TLS, access control | [Apache Web Server](../Apache-Web-Server/Readme.md) |
| [Lab-09-DNS-Server](Lab-09-DNS-Server.md) | Authoritative BIND9 master/slave zones | [Domain Name System (DNS)](../Domain-Name-System-DNS/Readme.md) |
| [Lab-10-DHCP-Server](Lab-10-DHCP-Server.md) | Scopes, reservations, allow/deny lists | [Dynamic Host Configuration Protocol (DHCP)](../Dynamic-Host-Configuration-Protocol-DHCP/Readme.md) |
| [Lab-11-Samba-File-Share](Lab-11-Samba-File-Share.md) | SMB shares, Windows interop, ACLs | [Samba SMB/CIFS Server](../Samba-SMB-CIFS-Server/Readme.md) |
| [Lab-12-NFS-Share](Lab-12-NFS-Share.md) | NFSv4 exports, mount options, idmapd | [NFS Server](../NFS-Server/Readme.md) |
| [Lab-13-LDAP-Authentication](Lab-13-LDAP-Authentication.md) | OpenLDAP + SSSD centralized auth | [LDAP Server](../LDAP-Server/Readme.md) |
| [Lab-14-Squid-Proxy](Lab-14-Squid-Proxy.md) | Forward proxy, ACLs, SSL bump basics | [Proxy Server (Squid)](../Proxy-Server-Squid/Readme.md) |
| [Lab-15-PXE-Boot](Lab-15-PXE-Boot.md) | TFTP + DHCP-chained network install | [TFTP & PXE Boot Server](../TFTP-and-PXE-Boot-Server/Readme.md) |
| [Lab-16-OpenWrt-Router](Lab-16-OpenWrt-Router.md) | OpenWrt gateway, VLANs, inter-VLAN routing | [Router & Firewall OS (OpenWrt)](../Router-and-Firewall-OS-OpenWrt/Readme.md) |
| [Lab-17-VPN-WireGuard](Lab-17-VPN-WireGuard.md) | Point-to-site WireGuard tunnel | [Network Configuration](../Network-Configuration/Readme.md) |
| [Lab-18-Monitoring-Stack](Lab-18-Monitoring-Stack.md) | Prometheus + Grafana + node_exporter | [Monitoring](../Monitoring/Readme.md) |
| [Lab-19-Performance-Tuning](Lab-19-Performance-Tuning.md) | Baselining, `tuned`, sysctl, I/O scheduler | [Performance & Tuning](../Performance-and-Tuning/Readme.md) |
| [Lab-20-Backup-and-Restore](Lab-20-Backup-and-Restore.md) | rsync/tar/restic strategy, DR drill | [File System & Disk Management](../File-System-and-Disk-Management/Readme.md) |

> [!NOTE]
> **📸 Screenshot**
> _Capture: your hypervisor's VM list/inventory view showing the lab VMs (e.g. `dns01`, `dhcp01`, `web01`) running side by side, to document your personal lab topology._

## How to Use These Labs

- **Snapshot before you start.** Take a hypervisor-level snapshot of each VM immediately after OS install and again before every lab that changes system state (partitioning, PAM, firewall, `/etc/fstab`). Name snapshots predictably, e.g. `base-clean`, `pre-lab06-ssh`.
- **Reset on failure, don't debug forever.** If a lab step leaves a service unbootable or a config unrecoverable and you're past your learning goal for that step, revert to the pre-lab snapshot rather than burning hours forensically fixing it — that's what snapshots are for.
- **Grade yourself against Validation.** Every lab's `## Validation` section lists numbered, concrete checks with expected output. Treat a lab as "done" only when every validation step passes on a fresh run, not just once during the messy first attempt.
- **Keep a lab log.** Note which distro/version you used, any deviation from the written steps, and IPs/hostnames you assigned — later labs (DNS, DHCP, LDAP) reference earlier ones and you'll want your own addressing scheme handy.
- **Re-run idempotently where possible.** Prefer configuration you can re-apply safely (`systemctl restart` after edits, not ad hoc one-off commands) so a lab can be redone after a snapshot revert without manual cleanup.

## Requirements

Shared baseline for the lab VMs referenced across this collection (individual labs list their own topology and any extra VMs/resources):

| Role | OS Family | vCPU | RAM | Disk | Network |
|---|---|---|---|---|---|
| Base template | RHEL-family (Rocky/Alma 9) or Debian-family (Debian 12 / Ubuntu 22.04+) | 1-2 | 1-2 GB | 20 GB | NAT + internal/host-only lab segment |
| Server role VMs (DNS, DHCP, Apache, Samba, NFS, LDAP, Squid, monitoring) | same as base template | 1-2 | 1-2 GB | 20 GB | Static IP on internal lab segment |
| Router/firewall VM (Lab 16) | OpenWrt (x86_64) | 1 | 256-512 MB | 1-4 GB | WAN (NAT) + LAN/VLAN trunk |
| Client VM (for testing shares, DHCP leases, VPN, DNS resolution) | any desktop-capable Linux/Windows | 1-2 | 2 GB | 20 GB | Internal lab segment |

> [!IMPORTANT]
> **Two distro families, one skill set**
> Labs show both RHEL-family (`dnf`, `firewalld`, `/etc/sysconfig/network-scripts`) and Debian-family (`apt`, `ufw`/`nftables`, Netplan) commands where package names or paths diverge. Pick one family for your own build-out and stay consistent — mixing tooling mid-lab is a common source of confusing errors.

## References

- Red Hat Enterprise Linux 9 documentation — https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9
- Debian Administrator's Handbook — https://debian-handbook.info/
- CIS Benchmarks (Distribution Independent Linux, RHEL, Debian/Ubuntu) — https://www.cisecurity.org/cis-benchmarks
- Arch Linux Wiki (service-specific deep dives, distro-agnostic) — https://wiki.archlinux.org/

## Related Notes

- [Users, Groups & Permissions](../Users-Groups-and-Permissions/Readme.md)
- [SSH Secure Shell Server](../SSH-Secure-Shell-Server/Readme.md)
- [Security, Firewall & Monitoring](../Security-Firewall-and-Monitoring/Readme.md)
- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
