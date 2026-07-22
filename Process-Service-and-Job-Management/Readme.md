# Process, Service and Job Management

Processes, memory, systemd services and timers, and scheduled jobs with cron.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Keeping workloads running and scheduled. This module covers process and memory management, controlling services with systemd (start/enable/status and unit files), running custom scripts at boot/shutdown/login, scheduling with cron, and the modern alternative of systemd timers.

## Learning Objectives

By the end of this module you will be able to:

- Inspect and control processes and interpret memory usage
- Manage services and create unit files with systemctl
- Schedule recurring work with cron and systemd timers

## Topics Covered

This module contains **9 notes**.

| Note | Topic |
| --- | --- |
| [Cron-Jobs-in-Linux](Cron-Jobs-in-Linux.md) | Cron Jobs in Linux |
| [GRUB2-Bootloader-Configuration](GRUB2-Bootloader-Configuration.md) | GRUB2 Bootloader Configuration |
| [Linux-Boot-Process](Linux-Boot-Process.md) | Linux Boot Process |
| [Memory-Management-in-Linux](Memory-Management-in-Linux.md) | Memory Management in Linux |
| [Process-Management-in-Linux](Process-Management-in-Linux.md) | Process Management in Linux |
| [Running-Custom-Scripts-on-Shutdown-Boot-Login-and-Logout-with-Systemd](Running-Custom-Scripts-on-Shutdown-Boot-Login-and-Logout-with-Systemd.md) | Running Custom Scripts on Shutdown Boot Login and Logout with Systemd |
| [Service-Management-in-Linux](Service-Management-in-Linux.md) | Service Management in Linux |
| [Systemd-Targets-and-Rescue-Mode](Systemd-Targets-and-Rescue-Mode.md) | Systemd Targets and Rescue Mode |
| [Systemd-Timers-in-Linux](Systemd-Timers-in-Linux.md) | Systemd Timers in Linux |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Prefer systemd units over ad-hoc rc scripts for lifecycle, logging, and dependencies
- Use systemd timers when you need calendar precision, logging, and dependency ordering
- Keep cron jobs idempotent and log their output

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Run services under dedicated unprivileged accounts and systemd sandboxing directives
- Restrict who can edit crontabs via `/etc/cron.allow`
- Avoid secrets on the command line where they appear in the process list

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Service fails to start | Read `systemctl status` and `journalctl -u <unit>` for the exact error |
| Cron job never runs | Check the environment/PATH, the user's crontab, and mail/log for errors |

## References

- [systemd documentation](https://www.freedesktop.org/wiki/Software/systemd/)
- [crontab(5) man page](https://man7.org/linux/man-pages/man5/crontab.5.html)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Performance & Tuning](../Performance-and-Tuning/Readme.md) — analyzing the processes and services managed here
- [Automation](../Automation/Readme.md) — scheduling work with cron and systemd timers
- [Shell Scripting](../Shell-Scripting/Readme.md) — related module
- [Security, Firewall and Monitoring](../Security-Firewall-and-Monitoring/Readme.md) — related module
- [Introduction to Linux](../Introduction-to-Linux/Readme.md) — related module
