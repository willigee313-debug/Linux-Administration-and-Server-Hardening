# Introduction to Linux

History, distributions, installation, and first-login orientation for Linux server administration.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

This module establishes the groundwork for the course: what Linux and UNIX are, how the major distribution families differ (Debian/Ubuntu vs Red Hat/CentOS/Rocky/Alma), and how to perform a clean base installation you will harden throughout the rest of the curriculum. It orients you in the filesystem, the login methods, and the administrator mindset used in every later module.

## Learning Objectives

By the end of this module you will be able to:

- Explain the relationship between UNIX, GNU, and Linux and pick a distribution for a given workload
- Perform a minimal Debian and CentOS Stream installation suitable for a server
- Log in locally and remotely and identify the files and services that define a fresh system
- Describe the server-hardening lifecycle applied across this course

## Topics Covered

This module contains **7 notes**.

| Note | Topic |
| --- | --- |
| [CentOS-Stream-Installation](CentOS-Stream-Installation.md) | CentOS Stream Installation |
| [Debian](Debian.md) | Debian |
| [Debian-System-Setup](Debian-System-Setup.md) | Debian System Setup |
| [Linux-Administration-Server-Hardening](Linux-Administration-Server-Hardening.md) | Linux Administration Server Hardening |
| [Linux-and-Unix](Linux-and-Unix.md) | Linux and Unix |
| [Login-Methods-in-Linux](Login-Methods-in-Linux.md) | Login Methods in Linux |
| [System-Localization-and-Time-Zones](System-Localization-and-Time-Zones.md) | System Localization and Time Zones |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Install to a minimal base and add only the packages a role requires
- Prefer LTS/stable release trains for production servers
- Snapshot a clean baseline VM before hardening so changes are reversible

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Set a strong root password and create an unprivileged admin user during install
- Disable unused install-time services and remove the desktop stack on servers
- Record the exact ISO checksum and release to make the build reproducible

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Installer cannot find the network mirror | Verify DNS/gateway in the installer network step; fall back to a local mirror |
| System boots to emergency mode | Check `/etc/fstab` entries and the root filesystem UUID |

## References

- [Debian Administrator's Handbook](https://debian-handbook.info/)
- [Red Hat Enterprise Linux docs](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux)
- [Arch Wiki: Installation guide](https://wiki.archlinux.org/title/Installation_guide)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Virtualization](../Virtualization/Readme.md) — the hypervisor layer hosting the course lab VMs
- [Containers](../Containers/Readme.md) — lightweight OS-level virtualization
- [Package Management](../Package-Management/Readme.md) — related module
- [Users, Groups and Permissions](../Users-Groups-and-Permissions/Readme.md) — related module
- [Linux Basic Commands](../Linux-Basic-Commands/Readme.md) — related module
