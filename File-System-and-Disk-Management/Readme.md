# File System and Disk Management

Partitions, filesystems, mounting, LVM, RAID, quotas, and swap.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Storage administration end to end: inspecting disks, partitioning, creating and permanently mounting filesystems, flexible volume management with LVM, redundancy with software RAID, per-user disk quotas, and swap sizing and extension. These skills keep servers resilient and their storage manageable as needs grow.

## Learning Objectives

By the end of this module you will be able to:

- Partition disks and create/mount filesystems persistently via /etc/fstab and UUIDs
- Provision flexible storage with LVM (PV/VG/LV) and resize online where supported
- Build and monitor software RAID and enforce disk quotas

## Topics Covered

This module contains **8 notes**.

| Note | Topic |
| --- | --- |
| [Disk-Management](Disk-Management.md) | Disk Management |
| [Disk-Quota-Management-in-Linux](Disk-Quota-Management-in-Linux.md) | Disk Quota Management in Linux |
| [Disk-Usage-Check](Disk-Usage-Check.md) | Disk Usage Check |
| [Disk-and-Partition-Management](Disk-and-Partition-Management.md) | Disk and Partition Management |
| [Logical-Volume-Manager(LVM)](Logical-Volume-Manager(LVM).md) | Logical Volume Manager(LVM) |
| [Permanent-Mounting-of-Partitions-in-Linux](Permanent-Mounting-of-Partitions-in-Linux.md) | Permanent Mounting of Partitions in Linux |
| [RAID-in-Linux](RAID-in-Linux.md) | RAID in Linux |
| [Swap-Extend](Swap-Extend.md) | Swap Extend |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Mount by UUID (not device name) so reordering disks does not break boot
- Leave headroom in volume groups so logical volumes can grow
- Test RAID rebuilds and quota policy before relying on them in production

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Use separate partitions with `nodev,nosuid,noexec` for `/tmp`, `/var`, and removable media where practical
- Consider LUKS full-disk or per-volume encryption for data at rest

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| System will not boot after fstab edit | Boot to rescue, comment the bad line, and correct the UUID/options |
| LVM shows less space than expected | The volume may not be extended onto the new PV; run `vgextend`/`lvextend` then grow the filesystem |

## References

- [Arch Wiki: LVM](https://wiki.archlinux.org/title/LVM)
- [Linux RAID wiki](https://raid.wiki.kernel.org/)
- [Red Hat: Managing file systems](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_file_systems/)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Virtualization](../Virtualization/Readme.md) — storage pools and virtual disks build on LVM and partitions
- [Performance & Tuning](../Performance-and-Tuning/Readme.md) — disk I/O analysis and tuning
- [Introduction to Linux](../Introduction-to-Linux/Readme.md) — related module
- [Process, Service and Job Management](../Process-Service-and-Job-Management/Readme.md) — related module
- [Security, Firewall and Monitoring](../Security-Firewall-and-Monitoring/Readme.md) — related module
