# NFS Server

UNIX/Linux file sharing with NFS: exports, mount options, Kerberos, and hardening.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Native file sharing between UNIX/Linux systems with NFS. This module covers configuring exports, client mounting and mount options, strengthening authentication with Kerberos, and hardening an NFS deployment against the weaknesses of host-based trust.

## Learning Objectives

By the end of this module you will be able to:

- Configure NFS exports and mount them from clients with appropriate options
- Choose mount options for performance, consistency, and safety
- Strengthen NFS with Kerberos and hardening controls

## Topics Covered

This module contains **6 notes**.

| Note | Topic |
| --- | --- |
| [NFS-Client-Setup-and-Mounting](NFS-Client-Setup-and-Mounting.md) | NFS Client Setup and Mounting |
| [NFS-Exports-Configuration](NFS-Exports-Configuration.md) | NFS Exports Configuration |
| [NFS-Mount-Options](NFS-Mount-Options.md) | NFS Mount Options |
| [NFS-Security-Hardening](NFS-Security-Hardening.md) | NFS Security Hardening |
| [NFS-with-Kerberos](NFS-with-Kerberos.md) | NFS with Kerberos |
| [Network-File-System-(NFS)-Server](Network-File-System-(NFS)-Server.md) | Network File System (NFS) Server |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Export the narrowest paths to the narrowest client ranges
- Use `root_squash` and appropriate `sync`/`async` semantics deliberately
- Prefer NFSv4 with Kerberos over legacy host-trust NFSv3 where possible

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Restrict exports by client subnet and enable `root_squash`
- Use Kerberos (`sec=krb5p`) for authentication and encryption on untrusted networks
- Firewall NFS/rpcbind ports to trusted clients only

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Mount hangs or is stale | Check network reachability, rpcbind/portmap, and matching NFS versions |
| Permission denied on an exported share | Verify export options, UID/GID mapping, and squash settings |

## References

- [Linux NFS documentation](https://linux-nfs.org/)
- [exports(5) man page](https://man7.org/linux/man-pages/man5/exports.5.html)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Samba (SMB/CIFS) Server](../Samba-SMB-CIFS-Server/Readme.md) — related module
- [LDAP Server](../LDAP-Server/Readme.md) — related module
- [Users, Groups and Permissions](../Users-Groups-and-Permissions/Readme.md) — related module
