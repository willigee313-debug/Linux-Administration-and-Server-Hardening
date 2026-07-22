# RAID in Linux

RAID (Redundant Array of Independent Disks) groups multiple physical disk drives into a single logical unit to optimize for **performance**, **redundancy**, or both.

## Overview

RAID lets a set of physical disks present themselves to the operating system as one logical device. Depending on the **RAID level** chosen, the array trades capacity for speed (striping), safety (mirroring/parity), or a blend of both. On Linux, RAID is most commonly implemented in software with the `mdadm` (multiple-device administration) utility, which manages arrays exposed as `/dev/mdN` block devices.

Software RAID via `mdadm` is hardware-independent, portable between systems, and requires no dedicated RAID controller — making it the default choice for most Linux servers and lab environments.

> [!NOTE]
> **RAID is not a backup**
> RAID protects against **disk failure**, not against accidental deletion, filesystem corruption, ransomware, or site loss. Always maintain independent backups in addition to any RAID redundancy.

## Concepts

### Common RAID levels in Linux

| RAID Level | Description                               | Minimum Drives | Fault Tolerance          | Use Case                                      |
|------------|-----------------------------------------|----------------|--------------------------|-----------------------------------------------|
| RAID 0     | Disk striping without redundancy        | 2              | None - data loss if one drive fails | High-speed access (video editing, gaming)     |
| RAID 1     | Disk mirroring                          | 2              | 1 drive failure           | Critical data, OS boot drives                   |
| RAID 5     | Block-level striping with distributed parity | 3          | 1 drive failure           | File servers, general-purpose storage          |
| RAID 6     | Like RAID 5 but with double parity      | 4              | 2 drive failures          | Large capacity storage, critical systems       |
| RAID 10    | Stripe of mirrors (RAID 1+0)            | 4              | 1 drive per mirrored pair | Databases, high-performance applications       |

> [!TIP]
> **Choosing a level**
> Use **RAID 1** for OS/boot drives, **RAID 10** for databases and write-heavy workloads, **RAID 5/6** for capacity-oriented file storage, and **RAID 0** only for scratch data you can afford to lose.

## Architecture

The diagram below shows how `mdadm` assembles member partitions into logical `/dev/mdN` arrays that are then formatted and mounted like any ordinary block device.

```mermaid
flowchart TD
    subgraph Physical Disks
      D1[/dev/sdb1]
      D2[/dev/sdc1]
      D3[/dev/sdd1]
    end
    D1 --> M[mdadm]
    D2 --> M
    D3 --> M
    M --> MD[/dev/md0 logical array]
    MD --> FS[mkfs.xfs]
    FS --> MP[Mount point /d1]
```

## Commands

### Creating RAID arrays

- Display help for `mdadm`:

```bash
mdadm --help
```

- Create RAID 0 (striping) with two devices `/dev/sdb1` and `/dev/sdc1`:

```bash
mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sdb1 /dev/sdc1
```

- Alternatively, use `--level=striping` (equivalent to RAID 0):

```bash
mdadm --create /dev/md0 --level=striping --raid-devices=2 /dev/sdb1 /dev/sdc1
```

- Create RAID 1 (mirroring):

```bash
mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdd1 /dev/sde1
```

- Alternatively, use `--level=mirror` (alias for RAID 1):

```bash
mdadm --create /dev/md1 --level=mirror --raid-devices=2 /dev/sdd1 /dev/sde1
```

- Create RAID 5 (striping with parity):

```bash
mdadm --create /dev/md5 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
```

- Create RAID 10 (striped mirrors):

```bash
mdadm --create /dev/md10 --level=10 --raid-devices=4 /dev/sdb /dev/sdc /dev/sdd /dev/sdf
```

### Checking RAID status

- View detailed RAID device information:

```bash
mdadm --detail /dev/md0
```

```bash
mdadm --detail /dev/md1
```

- Check overall RAID array status:

```bash
cat /proc/mdstat
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: Terminal output of cat /proc/mdstat showing active md0 array, RAID level, and member devices with their status flags_

### Filesystem and mounting

- Format RAID devices with XFS:

```bash
mkfs.xfs /dev/md0
```

- Mount RAID arrays:

```bash
mount /dev/md0 /d1/
```

```bash
mount /dev/md1 /d2/
```

- Check disk usage:

```bash
df -h
```

- Display block device UUIDs:

```bash
blkid
```

### Unmounting and stopping RAID arrays

- Unmount RAID arrays:

```bash
umount /dev/md0
```

```bash
umount /dev/md1
```

- Stop RAID arrays:

```bash
mdadm --stop /dev/md0
```

```bash
mdadm --stop /dev/md1
```

- Verify RAID device status after stopping:

```bash
mdadm --detail /dev/md0
```

```bash
mdadm --detail /dev/md1
```

- Additional queries:

> Block device info

```bash
mdadm -Db /dev/md0
```

> Query RAID type/info

```bash
mdadm -Q /dev/md0
```

### Reassembling RAID arrays

- Assemble all RAID arrays based on config:

```bash
mdadm --assemble --scan
```

- Confirm arrays are assembled and running:

```bash
cat /proc/mdstat
```

## Configuration

### Persisting RAID configuration

Without a saved configuration, an array may not reassemble with the same name after a reboot. Record the array definition in `mdadm.conf` and rebuild the initramfs so it is available early in boot.

- Save RAID config to persist after reboot:

```bash
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
```

- Update initramfs (Debian/Ubuntu):

```bash
update-initramfs -u
```

> [!IMPORTANT]
> **Also update /etc/fstab**
> Persisting the array definition only reassembles the device — add the corresponding mount entry (by UUID) to `/etc/fstab` so the filesystem mounts automatically. See [Permanent-Mounting-of-Partitions-in-Linux](Permanent-Mounting-of-Partitions-in-Linux.md).

## Best Practices

- Use partitions (not whole disks) as members where possible, and leave a small margin so a replacement disk of a slightly different size still fits.
- Always persist the configuration in `/etc/mdadm/mdadm.conf` and rebuild the initramfs after creating an array.
- Monitor array health continuously (`/proc/mdstat`, `mdadm --monitor`) and configure email alerts for degraded states.
- Keep independent backups — RAID is redundancy, not backup.
- For parity levels (5/6), account for the write penalty and prefer battery/UPS-backed systems to reduce the risk of the write hole.

## Security Considerations

- Restrict `mdadm` operations to root; array creation, stop, and assembly can destroy or expose data.
- Set correct permissions on mount points and consider hardened mount options (`nodev`, `nosuid`, `noexec`) on data arrays per CIS Benchmark guidance.
- For sensitive data, layer full-disk encryption (LUKS/dm-crypt) beneath or above the RAID array so a physically removed disk yields no readable data.
- Protect `/etc/mdadm/mdadm.conf` (root-owned, `0644`); a tampered config could cause the wrong devices to be assembled at boot.

## Troubleshooting

- **Array shows `[U_]` or a missing member in `/proc/mdstat`** — the array is degraded; identify the failed disk with `mdadm --detail`, then `mdadm --manage /dev/mdN --add /dev/sdX` a replacement to rebuild.
- **Array does not appear after reboot** — the definition was never saved; run `mdadm --detail --scan >> /etc/mdadm/mdadm.conf` and `update-initramfs -u`.
- **`mdadm --stop` reports the device is busy** — unmount the filesystem first (`umount`) and ensure no process holds it open.
- **Rebuild is slow** — this is expected for large parity arrays; monitor progress in `/proc/mdstat`.

## References

- `man mdadm`
- `man md`
- Linux RAID Wiki (kernel.org)
- Distribution documentation (Red Hat, Ubuntu Server Guide, Arch Wiki)

## Summary

Linux provides flexible RAID management through **software RAID** with `mdadm`. RAID improves system performance and/or data redundancy based on the selected level.

Key highlights:

- RAID 0 optimizes speed with no redundancy.
- RAID 1 duplicates data for safety.
- RAID 5 and RAID 6 employ parity for fault tolerance.
- RAID 10 combines striping and mirroring for performance and reliability.

Management requires root privileges to create, maintain, and monitor RAID arrays.

## Related

- [Disk-Management](Disk-Management.md) — overall disk administration
- [Logical-Volume-Manager(LVM)](Logical-Volume-Manager(LVM).md) — pair LVM with RAID for flexible volumes
- [Disk-and-Partition-Management](Disk-and-Partition-Management.md) — prepare member disks and partitions
- [Permanent-Mounting-of-Partitions-in-Linux](Permanent-Mounting-of-Partitions-in-Linux.md) — mount the array at boot
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
