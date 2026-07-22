# FTP Server (vsftpd)

Secure file transfer with vsftpd: users, chroot, anonymous access, and TLS.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Running an FTP service with vsftpd. This module covers user home-directory access, restricting logins to selected users, anonymous access, path and chroot configuration, root-login handling, and encrypting transfers with TLS (FTPS) — with an emphasis on doing FTP safely or choosing SFTP where appropriate.

## Learning Objectives

By the end of this module you will be able to:

- Install and configure vsftpd for authenticated and anonymous access
- Confine users to their home directories with chroot
- Encrypt control and data channels with TLS (FTPS)

## Topics Covered

This module contains **7 notes**.

| Note | Topic |
| --- | --- |
| [Access-User-Home-Directory-on-FTP-Server-using-vsftpd](Access-User-Home-Directory-on-FTP-Server-using-vsftpd.md) | Access User Home Directory on FTP Server using vsftpd |
| [Allow-Root-Login-on-vsftpd](Allow-Root-Login-on-vsftpd.md) | Allow Root Login on vsftpd |
| [Anonymous-FTP-Access-Configuration](Anonymous-FTP-Access-Configuration.md) | Anonymous FTP Access Configuration |
| [FTP-Client-Usage](FTP-Client-Usage.md) | FTP Client Usage |
| [FTP-Path-Configuration-in-vsftpd](FTP-Path-Configuration-in-vsftpd.md) | FTP Path Configuration in vsftpd |
| [Login-with-Selected-Users-on-vsftpd](Login-with-Selected-Users-on-vsftpd.md) | Login with Selected Users on vsftpd |
| [TLS-Encryption-on-FTP](TLS-Encryption-on-FTP.md) | TLS Encryption on FTP |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Prefer SFTP (over SSH) for most needs; use FTPS only when a real FTP client is required
- Chroot users to their home directory to prevent traversal
- Disable anonymous write unless there is a hardened, isolated drop directory

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Never allow FTP root login; keep it disabled
- Require explicit TLS and disable plaintext logins on untrusted networks
- Restrict passive-mode port ranges and open only those in the firewall

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Passive-mode transfers hang | Open and advertise the passive port range in vsftpd and the firewall |
| Chrooted user login fails | The chroot directory must not be writable by the user (vsftpd security check) |

## References

- [vsftpd documentation](https://security.appspot.com/vsftpd.html)
- [vsftpd.conf(5) man page](https://man.archlinux.org/man/vsftpd.conf.5)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [SSH Secure Shell Server](../SSH-Secure-Shell-Server/Readme.md) — related module
- [Security, Firewall and Monitoring](../Security-Firewall-and-Monitoring/Readme.md) — related module
- [Samba (SMB/CIFS) Server](../Samba-SMB-CIFS-Server/Readme.md) — related module
