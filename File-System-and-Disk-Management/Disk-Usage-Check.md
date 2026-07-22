# Disk Usage Check

## Overview

When a filesystem fills up, services fail to write, logs stop rotating, and the system can become unstable. This note collects the essential commands for **checking free space** (`df`) and **finding what is consuming it** (`du`, `ncdu`), with a focus on quickly locating the largest directories under `/` when the root filesystem is near capacity.

> [!TIP]
> Two tools cover most cases: `df -h` answers "how full is each filesystem?" and `du -h --max-depth=1 <dir> | sort -hr` answers "which directory is eating the space?".

## Concepts

`df` and `du` measure space in fundamentally different ways, which is why their numbers can disagree:

| Tool | Data source | Reports | Caveat |
| :-- | :-- | :-- | :-- |
| `df` | Filesystem superblock (allocation accounting) | Space allocated on the block device, including files that are unlinked but still held open | Includes reserved blocks and open-but-deleted files |
| `du` | Walks the directory tree, sums per-file usage | Space reachable through directory entries | Misses deleted-but-open files; can double-count hard links unless `-l` is used |

> [!NOTE]
> When `df` reports a filesystem as full but `du` accounts for far less, a process is almost always still holding a **deleted file** open. The inode's space is not released until the last file descriptor closes — see [Troubleshooting](#troubleshooting).

## Commands

### 1. Check overall disk usage

```bash
df -h /
```

#### Example Output

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   32G   16G  67% /
```

**Explanation**

| Field | Meaning |
| --- | --- |
| Size | Total disk size |
| Used | Used space |
| Avail | Free space |
| Use% | Percentage used |

### 2. Check disk usage of folders inside `/`

```bash
du -sh /*
```

#### Example

```text
4.0K    /bin
120M    /boot
8.2G    /home
12G     /var
3.1G    /usr
```

### 3. Find largest directories

```bash
du -h --max-depth=1 / | sort -hr
```

This lists directories inside `/` sorted by **largest size**.

### 4. Check all mounted filesystems

```bash
df -h
```

Shows disk usage for **all partitions**.

### 5. Interactive disk usage tool (recommended)

Install **ncdu**:

```bash
apt install ncdu
```

Run:

```bash
ncdu /
```

This opens an **interactive disk usage explorer** where you can navigate the directory tree and delete large items in place.

> [!NOTE]
> **📸 Screenshot**
> _Capture: ncdu terminal UI showing directories under / ranked by size with a navigable list and percentage bars_

## Examples

### Quick command (most useful)

```bash
du -h --max-depth=1 / | sort -hr
```

Useful when `/` disk is almost **100% full**.

> [!WARNING]
> Running `du` across the whole root filesystem also traverses pseudo-filesystems and network mounts. Add `-x` (`du -x -h --max-depth=1 /`) to stay on one filesystem and avoid scanning `/proc`, `/sys`, or NFS shares.

## Best Practices

- **Check inodes as well as blocks.** A filesystem can report free space yet still refuse writes when inodes are exhausted (common with many tiny files): `df -i`.
- **Reclaim safely.** Prefer rotating/compressing logs (`journalctl --vacuum-size=200M`, logrotate) and clearing package caches (`apt clean`, `dnf clean all`) over deleting unknown files.
- **Beware deleted-but-open files.** If `du` reports far less than `df`, a process is still holding a deleted file open — find it with `lsof | grep deleted` and restart the offending service to release the space.

## Security Considerations

- **Monitor disk usage proactively.** A full `/var` or `/` can silently stop `auditd`, `rsyslog`, and journald from writing, creating a logging blind spot that hides attacker activity. Alert on high `Use%` before it reaches 100%.
- **Investigate unexpected growth.** A sudden spike in `/tmp`, `/var/tmp`, or a web root can indicate dropped malware, staged exfiltration data, or log flooding — inspect the largest new directories rather than blindly deleting them.

## Troubleshooting

| Symptom | Likely Cause | Check / Fix |
| :-- | :-- | :-- |
| `df` shows full but `du` shows little used | Deleted files still held open | `lsof | grep deleted`, restart the process |
| "No space left on device" with free space in `df` | Inodes exhausted | `df -i`, delete/consolidate many small files |
| `du` on `/` is slow or huge | Traversing pseudo/network filesystems | Use `du -x` to stay on one filesystem |

## References

- `man 1 df`, `man 1 du`, `man 1 ncdu`
- Ubuntu Server Guide — Storage / disk usage

## Related

- [Disk-Management](Disk-Management.md) — overall disk administration
- [Disk-and-Partition-Management](Disk-and-Partition-Management.md) — partition layout behind usage
- [Disk-Quota-Management-in-Linux](Disk-Quota-Management-in-Linux.md) — enforce per-user/group limits
- [Memory-Management-in-Linux](../Process-Service-and-Job-Management/Memory-Management-in-Linux.md) — companion resource-usage reference
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
