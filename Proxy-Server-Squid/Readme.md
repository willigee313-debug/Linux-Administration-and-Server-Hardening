# Proxy Server (Squid)

Forward and transparent proxying with Squid: ACLs, authentication, and SSL bump.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Controlling and accelerating web traffic with Squid. This module covers server setup, access-control lists, NCSA basic authentication, transparent proxying, and SSL bump for inspecting HTTPS — the building blocks of an outbound web gateway with policy and caching.

## Learning Objectives

By the end of this module you will be able to:

- Deploy Squid as a forward and transparent proxy
- Enforce policy with ACLs and require user authentication
- Understand SSL bump and the trade-offs of HTTPS inspection

## Topics Covered

This module contains **5 notes**.

| Note | Topic |
| --- | --- |
| [Access-Control-List](Access-Control-List.md) | Access Control List |
| [Enable-Basic-NCSA-Authentication-in-Squid](Enable-Basic-NCSA-Authentication-in-Squid.md) | Enable Basic NCSA Authentication in Squid |
| [SSL-Bump-with-Squid-Proxy](SSL-Bump-with-Squid-Proxy.md) | SSL Bump with Squid Proxy |
| [Squid-Proxy-Server-Setup](Squid-Proxy-Server-Setup.md) | Squid Proxy Server Setup |
| [Squid-Transparent-Proxy](Squid-Transparent-Proxy.md) | Squid Transparent Proxy |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Write explicit ACLs and default-deny outbound where policy requires
- Log and review access for capacity and policy tuning
- Cache selectively; not all content benefits from caching

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Deploy SSL bump only with clear policy, user awareness, and a protected internal CA
- Require authentication for outbound access on managed networks
- Restrict who can reach the proxy port to prevent open-proxy abuse

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Clients bypass or cannot reach the proxy | Check ACL order, listening port, and transparent-redirect firewall rules |
| HTTPS sites break under SSL bump | The internal CA must be trusted by clients; some apps pin certificates and cannot be bumped |

## References

- [Squid documentation](https://www.squid-cache.org/Doc/)
- [Squid configuration manual](https://www.squid-cache.org/Doc/config/)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Apache Web Server](../Apache-Web-Server/Readme.md) — related module
- [Security, Firewall and Monitoring](../Security-Firewall-and-Monitoring/Readme.md) — related module
- [Network Configuration](../Network-Configuration/Readme.md) — related module
