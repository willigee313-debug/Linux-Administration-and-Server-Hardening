# SSH Secure Shell Server

OpenSSH server configuration, key authentication, access control, and hardening.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Secure remote administration with OpenSSH. This module covers sshd configuration, public/private key authentication and key generation, binding and port changes, controlling root login, and restricting access with allow/deny lists and TCP wrappers — the first service to harden on any internet-facing host.

## Learning Objectives

By the end of this module you will be able to:

- Configure sshd securely and generate/deploy SSH key pairs
- Enforce key-based authentication and disable password and root login
- Restrict access by user, IP, and interface

## Topics Covered

This module contains **10 notes**.

| Note | Topic |
| --- | --- |
| [Bind-SSH-To-A-Specific-IP-Address](Bind-SSH-To-A-Specific-IP-Address.md) | Bind SSH To A Specific IP Address |
| [Change-Default-SSH-Port](Change-Default-SSH-Port.md) | Change Default SSH Port |
| [Enable-Root-Login-via-SSH](Enable-Root-Login-via-SSH.md) | Enable Root Login via SSH |
| [Managing-Access-with-hosts.allow-and-hosts.deny](Managing-Access-with-hosts.allow-and-hosts.deny.md) | Managing Access with hosts.allow and hosts.deny |
| [Managing-IP-Allow-and-Deny-in-SSH](Managing-IP-Allow-and-Deny-in-SSH.md) | Managing IP Allow and Deny in SSH |
| [Prevent-Root-Login-via-SSH](Prevent-Root-Login-via-SSH.md) | Prevent Root Login via SSH |
| [SSH(Secure-Shell)-Server](SSH(Secure-Shell)-Server.md) | SSH(Secure Shell) Server |
| [SSH-Client-Tools](SSH-Client-Tools.md) | SSH Client Tools |
| [SSH-Keygen-Usage-and-SSH-Authentication-Setup](SSH-Keygen-Usage-and-SSH-Authentication-Setup.md) | SSH Keygen Usage and SSH Authentication Setup |
| [SSH-Public-and-Private-Key-Configuration](SSH-Public-and-Private-Key-Configuration.md) | SSH Public and Private Key Configuration |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Disable password authentication and root login; use keys plus sudo
- Change defaults thoughtfully and always keep a second session open when editing sshd_config
- Use `Match` blocks to scope policy per user/group/address

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Set `PermitRootLogin no` and `PasswordAuthentication no`
- Limit users with `AllowUsers`/`AllowGroups` and rate-limit with fail2ban
- Prefer certificate or hardware-backed keys for high-value hosts

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Locked out after an sshd edit | Use the still-open session or console; validate with `sshd -t` before reload |
| Key auth rejected | Check `~/.ssh` (700) and `authorized_keys` (600) permissions and ownership |

## References

- [OpenSSH manual](https://www.openssh.com/manual.html)
- [Mozilla OpenSSH guidelines](https://infosec.mozilla.org/guidelines/openssh)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Automation](../Automation/Readme.md) — Ansible drives configuration over SSH
- [Lab 06 — SSH Hardening](../Practical-Labs/Lab-06-SSH-Hardening.md) — hands-on for this module
- [Users, Groups and Permissions](../Users-Groups-and-Permissions/Readme.md) — related module
- [Security, Firewall and Monitoring](../Security-Firewall-and-Monitoring/Readme.md) — related module
- [Network Configuration](../Network-Configuration/Readme.md) — related module
