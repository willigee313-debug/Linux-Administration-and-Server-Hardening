# Swap Extend

"Swap Extend" refers to the process of increasing or extending the size of the swap space on a computer system, commonly a Linux system. Swap space is a critical component that acts as an overflow area for RAM, allowing systems to handle larger workloads by temporarily moving inactive memory pages to disk.

## Overview

When physical memory (RAM) is exhausted, the Linux kernel moves inactive pages to **swap space** on disk, freeing RAM for active processes. Swap also enables **hibernation** (suspend-to-disk), where the entire memory image is written to swap. Extending swap becomes necessary when workloads grow, when adding hibernation support, or when memory-intensive applications (databases, containers, build systems) begin to thrash.

Swap can live in three forms — a **logical volume**, a dedicated **partition**, or a **file** — and this note covers extending or creating each.

> [!NOTE]
> **Swap is slower than RAM**
> Swap on disk is orders of magnitude slower than RAM. Heavy sustained swapping ("thrashing") signals that the system needs more physical memory — swap is an overflow buffer, not a substitute for RAM.

## Concepts

### Methods to extend swap space

There are three common methods to extend swap space:

1. **Extend swap on an existing LVM2 logical volume:**
    - Disable swapping on the current swap logical volume using `swapoff`.
    - Resize the logical volume by the desired amount using `lvresize` or `lvextend`, ensuring sufficient free space in the volume group.
    - Format the new swap space with `mkswap`.
    - Enable the swap area again with `swapon`.
    - Verify the swap extension by checking active swap space with commands like `cat /proc/swaps` or `free`.
2. **Create a new swap partition:**
    - Use partitioning tools like `fdisk` or `parted` to create a new partition on an available disk.
    - Set the partition type to Linux swap (code `82`).
    - Format the partition as swap with `mkswap`.
    - Enable it using `swapon`.
3. **Create a new swap file:**
    - Create a dedicated swap file in the filesystem using `dd` or `fallocate`.
    - Set the file permissions to restrict access (typically `chmod 600`).
    - Format the file as swap with `mkswap`.
    - Activate with `swapon`.
    - Optionally add the swap file entry to `/etc/fstab` for persistence across reboots.

### Swap size recommendations based on RAM

| Installed RAM        | Recommended Swap                     | Rationale                                                        |
|----------------------|--------------------------------------|-----------------------------------------------------------------|
| Less than 2 GB       | **2× RAM**                           | Ample virtual memory for tight-RAM scenarios                    |
| 2 GB – 4 GB          | **Equal to RAM**                     | Balances disk usage without excessive swap                     |
| 4 GB – 8 GB          | **≈ RAM, or slightly less**          | Depends on workload                                             |
| More than 8 GB       | **At least 4 GB**                    | Large-RAM systems may use hibernation or memory-heavy apps     |

- Adjust swap size based on application requirements such as hibernation, database workload, or container usage.

> [!TIP]
> **Hibernation sizing**
> For hibernation (suspend-to-disk), swap must be **at least the size of RAM plus some overhead**, because the entire memory image is written to swap.

## Examples

### Practical example: extending swap on an LVM2 logical volume

To extend swap on an existing LVM2 logical volume named `/dev/VolGroup00/LogVol01` by 2GB:

- Disable the current swap:

```bash
swapoff -v /dev/VolGroup00/LogVol01
```

- Resize the logical volume (ensure sufficient free space in the volume group):

```bash
lvresize /dev/VolGroup00/LogVol01 -L +2G
```

- Format the extended volume as swap:

```bash
mkswap /dev/VolGroup00/LogVol01
```

- Enable swap on the new extended volume:

```bash
swapon -v /dev/VolGroup00/LogVol01
```

- Verify the swap space:

```bash
cat /proc/swaps
```

```bash
free -h
```

## Commands

### View swap and memory usage

- Human-readable:

```bash
free -h
```

- Megabytes:

```bash
free -m
```

- Gigabytes:

```bash
free -g
```

- Kilobytes:

```bash
free -k
```

### View active swap devices

- List active swap partitions and files:

```bash
swapon
```

- Summary of active swap:

```bash
swapon -s
```

### Create or manage swap partitions (example `/dev/sdd`)

- Launch partitioning tool:

```bash
fdisk /dev/sdd
```

- Inside fdisk:
    - List partitions with `p`
    - Create a new partition with `n`
    - Change partition type to Linux swap with `t`
    - Set type code to `82`
    - Write changes with `w`
- Format the partition as swap:

```bash
mkswap /dev/sdd1
```

- Enable the swap partition:

```bash
swapon /dev/sdd1
```

- Disable the swap partition if needed:

```bash
swapoff /dev/sdd1
```

### Adjust swap priority

- Assign priority (lower number = higher priority):

```bash
swapon -p 1 /dev/sdd1
```

- Change priority value as needed:

```bash
swapon -p 2 /dev/sdd1
```

> [!NOTE]
> **How priority works**
> Swap devices with **higher** priority values are used first. Devices sharing the same priority are used in a round-robin fashion, which can improve throughput when they sit on separate physical disks.

## Configuration

### Make swap persistent across reboots

- Edit `/etc/fstab` to add swap entries by UUID (find UUID with `blkid`):

```bash
vim /etc/fstab
```

- Example swap entries:

```bash
UUID=040d20c3-9b4d-457d-b581-62b251c192b6 swap swap defaults 0 0
UUID=80bc6709-1708-44b6-b633-fdfdebbf9b82 swap swap defaults 0 0
```

After editing `/etc/fstab`:

- Reboot the system or reload swap configuration:

```bash
reboot -f
```

or

```bash
swapon -a
```

### Summary of usage workflow

1. Check current swap with `free -h` and `swapon`.
2. Create or modify swap partitions or files with `fdisk` or file commands.
3. Format using `mkswap`.
4. Enable swap using `swapon`.
5. Adjust swap priority if running multiple swap devices.
6. Add swap entries to `/etc/fstab` for persistence.
7. Reboot or use `swapon -a` to activate swap during runtime.

## Best Practices

### Additional recommendations and tips

- Monitor swap usage regularly, as excessive swapping can degrade system performance.
- Use `vmstat` or `top` to observe swap activity and optimize configurations.
- On systems with SSDs, swap performance is improved but consider SSD wear.
- For hibernation, swap size should at least match RAM size plus some overhead.
- When adding swap files, ensure they are on fast disks and not heavily fragmented.
- Consider swapiness tuning via `/proc/sys/vm/swappiness` to adjust swap aggressiveness.

## Security Considerations

- Restrict swap-file permissions to `chmod 600` (root-only). A world-readable swap file could leak sensitive in-memory data (keys, passwords, tokens) that was paged out.
- Consider **encrypted swap** (dm-crypt / LUKS) on multi-user or portable systems so paged-out secrets are never written to disk in cleartext — a NIST/CIS data-at-rest recommendation.
- On high-security systems, evaluate disabling swap entirely (or setting `vm.swappiness=0`/`1`) to keep sensitive memory from touching disk, accepting the trade-off in available virtual memory.
- Keep `/etc/fstab` root-owned (`0644`) so swap definitions cannot be altered by unprivileged users.

## Troubleshooting

- **`swapoff` hangs or fails "Cannot allocate memory"** — the system lacks enough free RAM to hold the pages being evacuated from swap; free memory or add temporary swap before disabling.
- **New swap not active after reboot** — confirm the `/etc/fstab` entry and its UUID with `blkid`, then test with `swapon -a`.
- **Swap shows 0 despite being enabled** — verify the device was formatted with `mkswap` and enabled with `swapon`; check `swapon -s`.
- **Persistent high swap usage** — investigate memory-hungry processes with `top`/`vmstat`; tune `vm.swappiness` or add RAM.

## References

- `man swapon`
- `man mkswap`
- `man fstab`
- `Documentation/admin-guide/mm/` (Linux kernel memory-management docs)
- Distribution documentation (Red Hat, Ubuntu Server Guide, Arch Wiki)

## Related

- [Memory-Management-in-Linux](../Process-Service-and-Job-Management/Memory-Management-in-Linux.md) — swap supplements RAM
- [Logical-Volume-Manager(LVM)](Logical-Volume-Manager(LVM).md) — extend swap on logical volumes
- [Disk-Management](Disk-Management.md) — overall disk administration
- [Permanent-Mounting-of-Partitions-in-Linux](Permanent-Mounting-of-Partitions-in-Linux.md) — persist swap via `/etc/fstab`
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
