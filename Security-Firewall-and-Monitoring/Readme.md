# Security, Firewall and Monitoring

Host firewalls (iptables/firewalld/ufw), antivirus, packet capture, and boot protection.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Core defensive controls for a Linux host: packet-filtering firewalls with iptables, firewalld, and ufw; malware scanning with ClamAV; traffic inspection with tcpdump; and protecting the boot path by securing the GRUB bootloader and root-password recovery. Together these implement defense-in-depth at the host level.

## Learning Objectives

By the end of this module you will be able to:

- Design and apply firewall policy with iptables, firewalld, or ufw
- Capture and analyze traffic with tcpdump for troubleshooting and detection
- Protect the boot loader and understand root-password recovery risk

## Topics Covered

This module contains **13 notes**.

| Note | Topic |
| --- | --- |
| [ClamAV-Antivirus](ClamAV-Antivirus.md) | ClamAV Antivirus |
| [Firewall-Network-Security-Barrier](Firewall-Network-Security-Barrier.md) | Firewall Network Security Barrier |
| [Firewalld](Firewalld.md) | Firewalld |
| [IPTables-Configuration-and-Management](IPTables-Configuration-and-Management.md) | IPTables Configuration and Management |
| [Network-Operating-Systems-and-Security-Solutions](Network-Operating-Systems-and-Security-Solutions.md) | Network Operating Systems and Security Solutions |
| [nftables-Configuration-and-Management](nftables-Configuration-and-Management.md) | nftables Configuration and Management |
| [Reset-Root-Password-and-Protect-GRUB-Boot-Loader](Reset-Root-Password-and-Protect-GRUB-Boot-Loader.md) | Reset Root Password and Protect GRUB Boot Loader |
| [SELinux-Booleans-and-Ports](SELinux-Booleans-and-Ports.md) | SELinux Booleans and Ports |
| [SELinux-Contexts-and-File-Labeling](SELinux-Contexts-and-File-Labeling.md) | SELinux Contexts and File Labeling |
| [SELinux-Fundamentals](SELinux-Fundamentals.md) | SELinux Fundamentals |
| [SELinux-Troubleshooting](SELinux-Troubleshooting.md) | SELinux Troubleshooting |
| [TCPDump-Command](TCPDump-Command.md) | TCPDump Command |
| [Uncomplicated-Firewall(ufw)](Uncomplicated-Firewall(ufw).md) | Uncomplicated Firewall(ufw) |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Default-deny inbound and allow only required services
- Keep one consistent firewall front-end per host to avoid conflicting rule sets
- Baseline normal traffic so anomalies stand out

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Password-protect GRUB to prevent trivial single-user root recovery on physical/console access
- Log dropped packets for high-value segments
- Run AV/rootkit scans on file servers and mail gateways on a schedule

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Locked out after a firewall change | Use console/out-of-band; apply rules with a rollback timer when remote |
| firewalld and iptables rules conflict | Manage the firewall through a single front-end and reload cleanly |

## References

- [netfilter/iptables project](https://www.netfilter.org/)
- [firewalld documentation](https://firewalld.org/documentation/)
- [NIST SP 800-41 (firewall guidelines)](https://csrc.nist.gov/pubs/sp/800/41/r1/final)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Monitoring](../Monitoring/Readme.md) — logging, auditd, metrics and alerting extend this baseline
- [Enterprise Firewall (Project 07)](../Enterprise-Projects/Project-07-Enterprise-Firewall.md) — capstone applying these controls
- [SSH Secure Shell Server](../SSH-Secure-Shell-Server/Readme.md) — related module
- [Network Configuration](../Network-Configuration/Readme.md) — related module
- [Users, Groups and Permissions](../Users-Groups-and-Permissions/Readme.md) — related module
