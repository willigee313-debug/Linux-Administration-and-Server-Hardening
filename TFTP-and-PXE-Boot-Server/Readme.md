# TFTP and PXE Boot Server

Network booting and automated installs with TFTP and PXE.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Booting and installing systems over the network. This module covers the TFTP server and client and the PXE (Preboot eXecution Environment) boot server, which together with DHCP and DNS enable diskless boot and automated, repeatable OS deployment at scale.

## Learning Objectives

By the end of this module you will be able to:

- Configure a TFTP server to serve boot files
- Stand up a PXE environment integrated with DHCP for network boot
- Chain to an automated OS installer for repeatable deployment

## Topics Covered

This module contains **3 notes**.

| Note | Topic |
| --- | --- |
| [Preboot-eXecution-Environment(PXE)-Boot-Server](Preboot-eXecution-Environment(PXE)-Boot-Server.md) | Preboot eXecution Environment(PXE) Boot Server |
| [TFTP-Client](TFTP-Client.md) | TFTP Client |
| [Trivial-File-Transfer-Protocol(TFTP)-Server](Trivial-File-Transfer-Protocol(TFTP)-Server.md) | Trivial File Transfer Protocol(TFTP) Server |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Coordinate PXE with the authoritative DHCP server (next-server/filename options)
- Keep boot images and kickstart/preseed files version-controlled
- Isolate the provisioning network segment

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- TFTP is unauthenticated and plaintext — confine it to an isolated provisioning VLAN
- Restrict which hosts DHCP will PXE-boot
- Validate the integrity of served boot images

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Client does not PXE boot | Check DHCP next-server/filename, TFTP reachability, and firmware boot order |
| TFTP transfer fails | Verify the TFTP root, file permissions, and UDP port 69 through the firewall |

## References

- [Syslinux/PXELINUX wiki](https://wiki.syslinux.org/wiki/index.php?title=PXELINUX)
- [tftpd(8) man page](https://man.archlinux.org/man/tftpd.8)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Dynamic Host Configuration Protocol (DHCP)](../Dynamic-Host-Configuration-Protocol-DHCP/Readme.md) — related module
- [Domain Name System (DNS)](../Domain-Name-System-DNS/Readme.md) — related module
- [Network Configuration](../Network-Configuration/Readme.md) — related module
