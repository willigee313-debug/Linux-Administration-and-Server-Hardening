# Telnet and Remote Desktop Server

Graphical and legacy remote access: XRDP, VNC, NoMachine (and why not Telnet).

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Remote graphical and legacy access to Linux hosts. This module covers XRDP (RDP), VNC, and NoMachine for remote desktops, plus Telnet for legacy scenarios — with a strong emphasis on why plaintext protocols like Telnet must be avoided or tunneled and how to secure graphical remote access.

## Learning Objectives

By the end of this module you will be able to:

- Provide remote desktop access with XRDP, VNC, or NoMachine
- Understand the security posture of each remote-access method
- Recognize why Telnet is unsafe and what to use instead

## Topics Covered

This module contains **6 notes**.

| Note | Topic |
| --- | --- |
| [NoMachine](NoMachine.md) | NoMachine |
| [Remote-Desktop-Setup](Remote-Desktop-Setup.md) | Remote Desktop Setup |
| [Telnet-Server](Telnet-Server.md) | Telnet Server |
| [VNC-Server](VNC-Server.md) | VNC Server |
| [XRDP-Server-Configuration](XRDP-Server-Configuration.md) | XRDP Server Configuration |
| [XRDP-Server-Kali-Linux](XRDP-Server-Kali-Linux.md) | XRDP Server Kali Linux |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Prefer SSH-based access; tunnel VNC/graphical sessions over SSH
- Use NoMachine or XRDP with TLS for interactive desktops
- Restrict remote-desktop ports to management networks/VPN

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Never expose Telnet or plaintext VNC to untrusted networks — use SSH/TLS
- Require strong authentication and restrict source addresses
- Lock idle sessions and log remote-desktop access

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| XRDP shows a black screen | Session/desktop environment mismatch; configure the correct session in `.xsession` |
| VNC is slow or insecure | Tunnel over SSH and tune encoding/compression |

## References

- [xrdp project](https://github.com/neutrinolabs/xrdp)
- [TigerVNC](https://tigervnc.org/)
- [NoMachine documentation](https://www.nomachine.com/documents)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [SSH Secure Shell Server](../SSH-Secure-Shell-Server/Readme.md) — related module
- [Security, Firewall and Monitoring](../Security-Firewall-and-Monitoring/Readme.md) — related module
- [Network Configuration](../Network-Configuration/Readme.md) — related module
