# Users, Groups and Permissions

Account management, the permission model, special bits, ACLs, and sudo policy.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Access control is the heart of Linux security. This large module covers user and group account management (useradd/usermod/groupadd and the passwd/shadow/group/gshadow databases), the read/write/execute permission model, ownership, special permissions (SetUID, SetGID, sticky bit), POSIX ACLs, real vs effective UID, and fine-grained privilege delegation with sudo and the sudoers policy.

## Learning Objectives

By the end of this module you will be able to:

- Create and manage users and groups and understand the four account databases
- Apply and reason about permissions, ownership, and special bits
- Grant least-privilege administrative access with sudoers aliases and NOPASSWD scoping
- Use ACLs for access needs the classic model cannot express

## Topics Covered

This module contains **37 notes**.

| Note | Topic |
| --- | --- |
| [Access-Control-List(ACL)](Access-Control-List(ACL).md) | Access Control List(ACL) |
| [Account-Lockout-with-pam_faillock](Account-Lockout-with-pam_faillock.md) | Account Lockout with pam_faillock |
| [chgrp](chgrp.md) | chgrp |
| [chmod](chmod.md) | chmod |
| [chown](chown.md) | chown |
| [Command-Aliases-in-sudoers](Command-Aliases-in-sudoers.md) | Command Aliases in sudoers |
| [deluser](deluser.md) | deluser |
| [gpasswd](gpasswd.md) | gpasswd |
| [Group-File-Linux-Group-Account-File](Group-File-Linux-Group-Account-File.md) | Group File Linux Group Account File |
| [groupadd](groupadd.md) | groupadd |
| [groupdel](groupdel.md) | groupdel |
| [groupmod](groupmod.md) | groupmod |
| [Gshadow-File-Secure-Group-Access-File](Gshadow-File-Secure-Group-Access-File.md) | Gshadow File Secure Group Access File |
| [Host-Aliases-in-sudoers](Host-Aliases-in-sudoers.md) | Host Aliases in sudoers |
| [Linux-Permission-Assignment](Linux-Permission-Assignment.md) | Linux Permission Assignment |
| [Linux-Permissions](Linux-Permissions.md) | Linux Permissions |
| [PAM-Pluggable-Authentication-Modules](PAM-Pluggable-Authentication-Modules.md) | PAM (Pluggable Authentication Modules) |
| [passwd](passwd.md) | passwd |
| [Passwd-File-Linux-User-Account-File](Passwd-File-Linux-User-Account-File.md) | Passwd File Linux User Account File |
| [Password-Policy-with-pam_pwquality](Password-Policy-with-pam_pwquality.md) | Password Policy with pam_pwquality |
| [Privilege-Escalation-Example-Using](Privilege-Escalation-Example-Using.md) | Privilege Escalation Example Using |
| [Real-UID-vs-Effective-UID-in-Linux](Real-UID-vs-Effective-UID-in-Linux.md) | Real UID vs Effective UID in Linux |
| [Real-World-Examples-Combine-Aliases](Real-World-Examples-Combine-Aliases.md) | Real World Examples Combine Aliases |
| [Reset-the-Root-Password-in-CentOS-Stream-10](Reset-the-Root-Password-in-CentOS-Stream-10.md) | Reset the Root Password in CentOS Stream 10 |
| [Setgid(Set-Group-ID)](Setgid(Set-Group-ID).md) | Setgid(Set Group ID) |
| [Setuid(Set-User-ID)](Setuid(Set-User-ID).md) | Setuid(Set User ID) |
| [Shadow-File-Secure-User-Passwords-File](Shadow-File-Secure-User-Passwords-File.md) | Shadow File Secure User Passwords File |
| [Special-Permission](Special-Permission.md) | Special Permission |
| [Sticky-Bit](Sticky-Bit.md) | Sticky Bit |
| [su-and-sg](su-and-sg.md) | su and sg |
| [Sudo](Sudo.md) | Sudo |
| [Understanding-NOPASSWD-in-sudoers](Understanding-NOPASSWD-in-sudoers.md) | Understanding NOPASSWD in sudoers |
| [User-Aliases-in-sudoers](User-Aliases-in-sudoers.md) | User Aliases in sudoers |
| [User-and-Group-Management](User-and-Group-Management.md) | User and Group Management |
| [User-Management-with-useradd-and-adduser](User-Management-with-useradd-and-adduser.md) | User Management with useradd and adduser |
| [userdel](userdel.md) | userdel |
| [usermod](usermod.md) | usermod |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Grant the narrowest sudo rights that accomplish the task; avoid blanket `ALL=(ALL) ALL`
- Audit SetUID/SetGID binaries regularly (`find / -perm -4000`)
- Use groups for shared access instead of loosening file permissions

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Enforce strong password and aging policy via `/etc/login.defs` and PAM
- Keep `/etc/shadow` at mode 0640/0600 and never world-readable
- Validate sudoers edits with `visudo` to avoid locking out administration
- Prefer `sudo` over shared root logins for accountability and audit trails

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| User cannot sudo despite being added | Group membership requires a fresh login/session to take effect |
| Permission denied on a file the user should access | Check directory execute bits along the whole path and any ACL/`setfacl` entries |

## References

- [Red Hat: Managing users and groups](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/)
- [sudoers(5) man page](https://man7.org/linux/man-pages/man5/sudoers.5.html)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Containers](../Containers/Readme.md) — user namespaces and rootless containers extend these permissions
- [Security, Firewall and Monitoring](../Security-Firewall-and-Monitoring/Readme.md) — related module
- [SSH Secure Shell Server](../SSH-Secure-Shell-Server/Readme.md) — related module
- [Introduction to Linux](../Introduction-to-Linux/Readme.md) — related module
