# Samba (SMB/CIFS) Server

Windows-compatible file sharing with Samba: shares, users, and access control.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Interoperable file sharing with Windows clients using Samba. This module covers server setup, anonymous and authenticated shares, restricting shares to selected users, and shared common directories — the standard way to provide SMB/CIFS storage from a Linux server.

## Learning Objectives

By the end of this module you will be able to:

- Install and configure Samba shares for anonymous and authenticated access
- Map Samba accounts and restrict shares to specific users or groups
- Provide a shared common directory with correct permissions

## Topics Covered

This module contains **6 notes**.

| Note | Topic |
| --- | --- |
| [Anonymous-Samba-Share](Anonymous-Samba-Share.md) | Anonymous Samba Share |
| [Samba-SMB-CIFS-Server](Samba-SMB-CIFS-Server.md) | Samba SMB CIFS Server |
| [Samba-Server-Setup](Samba-Server-Setup.md) | Samba Server Setup |
| [Share-With-Selected-Users](Share-With-Selected-Users.md) | Share With Selected Users |
| [Shared-Common-Directories-With-Samba](Shared-Common-Directories-With-Samba.md) | Shared Common Directories With Samba |
| [Smb-Client-tools](Smb-Client-tools.md) | Smb Client tools |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Keep share definitions minimal and explicit about who may access them
- Use dedicated Samba accounts mapped to system users
- Separate public and private shares with distinct permission models

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Require SMB signing and disable the obsolete SMBv1 protocol
- Restrict shares by `valid users`/`hosts allow` and integrate with the host firewall
- Align UNIX permissions and Samba ACLs so neither silently over-grants

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Windows client cannot connect | Check SMB protocol version, credentials, and firewall ports 139/445 |
| Access denied despite correct password | Reconcile UNIX permissions, `valid users`, and the mapped account |

## References

- [Samba documentation](https://www.samba.org/samba/docs/)
- [smb.conf(5) man page](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [FTP Server (vsftpd)](../FTP-Server-VSFTPD/Readme.md) — related module
- [NFS Server](../NFS-Server/Readme.md) — related module
- [Users, Groups and Permissions](../Users-Groups-and-Permissions/Readme.md) — related module
