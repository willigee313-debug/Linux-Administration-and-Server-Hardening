# LDAP Server

Centralized directory and authentication with OpenLDAP: structure, LDIF, and TLS.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Centralized identity with OpenLDAP. This module covers the directory information tree and schema, LDIF for adding and modifying entries, client configuration for authentication, securing the directory with TLS, and hardening — enabling single-source-of-truth accounts across many hosts.

## Learning Objectives

By the end of this module you will be able to:

- Explain the LDAP directory structure, schema, and LDIF format
- Stand up an OpenLDAP server and configure clients to authenticate against it
- Secure directory traffic with TLS and apply hardening controls

## Topics Covered

This module contains **8 notes**.

| Note | Topic |
| --- | --- |
| [LDAP-Authentication](LDAP-Authentication.md) | LDAP Authentication |
| [LDAP-Client-Configuration](LDAP-Client-Configuration.md) | LDAP Client Configuration |
| [LDAP-Directory-Structure](LDAP-Directory-Structure.md) | LDAP Directory Structure |
| [LDAP-Security-Hardening](LDAP-Security-Hardening.md) | LDAP Security Hardening |
| [LDAP-over-TLS](LDAP-over-TLS.md) | LDAP over TLS |
| [LDIF-Files-and-Schema](LDIF-Files-and-Schema.md) | LDIF Files and Schema |
| [Ldap-Server-Setup](Ldap-Server-Setup.md) | Ldap Server Setup |
| [OpenLDAP-Overview](OpenLDAP-Overview.md) | OpenLDAP Overview |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Design the DIT and naming before loading data at scale
- Manage changes as version-controlled LDIF
- Use groups/roles in the directory to drive access decisions

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Require TLS (LDAPS/StartTLS) so credentials never cross the wire in cleartext
- Restrict anonymous binds and scope ACLs to least privilege
- Back up the directory and monitor bind failures

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Clients cannot authenticate | Check base DN, bind credentials, TLS trust, and nsswitch/PAM configuration |
| TLS handshake fails | Verify the server certificate chain and the client's CA trust store |

## References

- [OpenLDAP Administrator's Guide](https://www.openldap.org/doc/admin26/)
- [RFC 4511 (LDAP protocol)](https://www.rfc-editor.org/rfc/rfc4511)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Users, Groups and Permissions](../Users-Groups-and-Permissions/Readme.md) — related module
- [NFS Server](../NFS-Server/Readme.md) — related module
- [SSH Secure Shell Server](../SSH-Secure-Shell-Server/Readme.md) — related module
