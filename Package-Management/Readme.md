# Package Management

Installing and maintaining software with APT/dpkg and DNF/YUM/RPM.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Software lifecycle management on both major distribution families. This module covers Debian's dpkg and APT (apt, apt-get, apt-cache) and Red Hat's RPM with YUM and DNF, including repositories, GPG key verification, package signing, and offline installation — the foundation of a patchable, trustworthy system.

## Learning Objectives

By the end of this module you will be able to:

- Install, upgrade, query, and remove packages on Debian- and RHEL-family systems
- Add and trust repositories and verify package signatures with GPG
- Perform offline/air-gapped installation when a mirror is unavailable

## Topics Covered

This module contains **10 notes**.

| Note | Topic |
| --- | --- |
| [Advanced-Package-Tool(APT)](Advanced-Package-Tool(APT).md) | Advanced Package Tool(APT) |
| [CentOS-9-minimal-to-GUI-Installation](CentOS-9-minimal-to-GUI-Installation.md) | CentOS 9 minimal to GUI Installation |
| [DNF-Package-Manager](DNF-Package-Manager.md) | DNF Package Manager |
| [Debian-Package-Manager(dpkg)](Debian-Package-Manager(dpkg).md) | Debian Package Manager(dpkg) |
| [Package-Manager-in-Linux](Package-Manager-in-Linux.md) | Package Manager in Linux |
| [Red-Hat-Package-Manager(RPM)](Red-Hat-Package-Manager(RPM).md) | Red Hat Package Manager(RPM) |
| [Wine-Install-in-Centos-9](Wine-Install-in-Centos-9.md) | Wine Install in Centos 9 |
| [Yum(Yellowdog-Updater-Modified)](Yum(Yellowdog-Updater-Modified).md) | Yum(Yellowdog Updater Modified) |
| [Yum-Command-Reference](Yum-Command-Reference.md) | Yum Command Reference |
| [apt-cache-and-apt-get](apt-cache-and-apt-get.md) | apt cache and apt get |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Keep systems patched on a regular cadence; automate security updates where appropriate
- Only add repositories you trust and pin critical packages to avoid surprise upgrades
- Verify GPG keys out-of-band before importing them

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Never disable signature checking (`--nogpgcheck`/`[trusted=yes]`) on production
- Review changelogs for security fixes before deferring updates
- Remove unused packages to reduce attack surface

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| APT reports unauthenticated packages | Import the correct signing key; do not bypass with `--allow-unauthenticated` |
| DNF dependency resolution fails | Check enabled repos and module streams; clean the cache with `dnf clean all` |

## References

- [Debian APT documentation](https://wiki.debian.org/Apt)
- [DNF documentation](https://dnf.readthedocs.io/)
- [RPM documentation](https://rpm.org/documentation.html)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Containers](../Containers/Readme.md) — container images as an alternative packaging and distribution model
- [Introduction to Linux](../Introduction-to-Linux/Readme.md) — related module
- [Security, Firewall and Monitoring](../Security-Firewall-and-Monitoring/Readme.md) — related module
