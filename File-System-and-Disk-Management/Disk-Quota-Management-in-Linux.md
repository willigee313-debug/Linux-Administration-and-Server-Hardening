# Disk Quota Management in Linux

## Overview

Disk quotas allow administrators to **restrict and monitor disk space (blocks) and inode usage (file count)** for **users** and **groups** on a filesystem. This ensures that no single user consumes all storage resources — a key control on shared servers, mail hosts, and multi-tenant systems.

> [!NOTE]
> Quotas are enforced per **filesystem**, not globally. A quota on `/home` does nothing for files a user writes under `/tmp` on a different filesystem.

## Concepts

### Limits and thresholds

| Term | Meaning |
| :-- | :-- |
| **Blocks** | Disk space consumed (measured in 1 KB blocks by default) |
| **Inodes** | Number of files/objects owned |
| **Soft limit** | Warning threshold — may be exceeded temporarily during the grace period |
| **Hard limit** | Absolute ceiling — the kernel refuses further allocation once reached |
| **Grace period** | How long a user may stay above the soft limit before it is enforced like a hard limit |

### Enforcement workflow

```mermaid
flowchart LR
    A["Install quota tools"] --> B["Enable in /etc/fstab<br/>usrquota,grpquota"]
    B --> C["Remount"]
    C --> D["Create quota DB<br/>quotacheck (ext4)"]
    D --> E["quotaon"]
    E --> F["Set limits<br/>edquota"]
    F --> G["Monitor<br/>repquota / quota"]
```

## Configuration

### 1. Install quota tools

Required for enabling and managing disk quotas.

- Debian/Ubuntu

```bash
apt install quota
```

- RHEL/CentOS

```bash
yum install quota
```

- Fedora

```bash
dnf install quota
```

Inspect the installed package:

```bash
rpm -qa | grep quota
```

```bash
rpm -qi quota-4.09-4.el9.x86_64
```

```bash
rpm -qd quota-4.09-4.el9.x86_64
```

```bash
rpm -ql quota-4.09-4.el9.x86_64
```

### 2. Enable quota on the filesystem

Quotas must be **enabled at the filesystem level** using `/etc/fstab`.

```bash
vim /etc/fstab
```

Example entries:

```text
UUID=b57ee1c2-24fb-4a78-af57-e0b6ea5f7288  /d1  xfs  defaults,usrquota,grpquota  0 0
UUID=826163ad-d436-4f7b-b8ed-e803c43be46e  /d2  ext4  defaults,usrquota,grpquota  0 0
```

| Mount option | Purpose |
| :-- | :-- |
| `usrquota` | Enable user disk quotas |
| `grpquota` | Enable group disk quotas |

### 3. Remount the filesystem

Apply the changes without reboot:

```bash
mount -o remount /d1
```

Or remount all modified entries:

```bash
mount -a
```

### 4. Create quota database files

Quota files store usage statistics.

For **ext4**:

- Create user quota files

```bash
quotacheck -cum /d1
```

- Create group quota files

```bash
quotacheck -cgm /d1
```

Options:

| Option | Meaning |
| :-- | :-- |
| `-c` | Create new quota files |
| `-u` | Check user quotas |
| `-g` | Check group quotas |
| `-m` | Don't remount filesystem read-only |

For **XFS**, use `xfs_quota` (no `quotacheck` needed, since quota is built-in):

```bash
xfs_quota -x -c 'enable' /d1
```

```bash
xfs_quota -x -c 'report' /d1
```

### 5. Enable quotas

Turn on the quota engine:

```bash
quotaon -v /d1
```

- Enable quotas for all filesystems

```bash
quotaon -ap
```

To turn off:

```bash
quotaoff /d1
```

## Commands

### 6. Assign quotas to users

Edit quota limits interactively:

```bash
edquota -u username
```

Example quota edit:

```text
Disk quotas for user armour (uid 1000):
  Filesystem    blocks   soft   hard   inodes   soft   hard
  /dev/sdb1     200000 200000     0       90      0      0
```

- **Soft Limit:** Warning threshold (can exceed for grace period)
- **Hard Limit:** Absolute maximum (cannot exceed)
- **Blocks:** Disk space
- **Inodes:** Number of files

Set the grace period (how long the soft limit can be exceeded):

```bash
edquota -t
```

For scripted, non-interactive provisioning, use `setquota` instead of the editor-driven `edquota`. The four numeric fields are `block-soft block-hard inode-soft inode-hard`:

```bash
setquota -u armour 180000 200000 80 90 /d1
```

> [!TIP]
> `setquota` is the automation-friendly counterpart to `edquota` — ideal for onboarding scripts, configuration management (Ansible/Puppet), and applying identical limits to many users in a loop.

### 7. Monitoring quotas

Check a user's quota:

```bash
quota -u username
```

Full report for all users:

```bash
repquota -as
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: repquota -as output table listing each user with used blocks, soft and hard block limits, grace column, and inode usage_

## Examples

### 8. Example workflow (ext4, end to end)

- Format with ext4

```bash
mkfs.ext4 /dev/sdb1
```

- Enable quotas at the filesystem level

```bash
tune2fs -O quota /dev/sdb1
```

```bash
mount /dev/sdb1 /d1
```

```bash
echo "/dev/sdb1 /d1 ext4 defaults,usrquota,grpquota 0 0" >> /etc/fstab
```

```bash
mount -a
```

```bash
quotacheck -cugm /d1
```

```bash
quotaon /d1
```

- Assign block/inode quota

```bash
edquota -u armour
```

- Check all quotas

```bash
repquota -as
```

## ext4 vs XFS

| Aspect | ext4 | XFS |
| :-- | :-- | :-- |
| Quota database | External files created by `quotacheck` | Built into the filesystem metadata |
| Enable command | `quotaon` after `quotacheck` | `xfs_quota -x -c 'enable'` |
| Management tool | `edquota`, `repquota`, `quota` | `xfs_quota` (also supports `edquota`/`repquota`) |
| Project quotas | Limited | Native `pquota` support |

- **ext4:** Requires `quotacheck` to generate quota files.
- **xfs:** Quota support is built-in; use `xfs_quota`.

## Best Practices

- **Set both soft and hard limits** with a sensible grace period so users get warned before writes hard-fail.
- **Quota inodes as well as blocks.** A user can exhaust a filesystem's inode table with countless tiny files while staying well under the block limit.
- **Run `quotacheck` during low activity** (ideally with the filesystem quiescent) since it scans the whole filesystem; schedule periodic rechecks to correct drift.
- **Prefer XFS project quotas** for directory-tree limits (e.g. per-project or per-container storage) rather than per-user quotas.

## Security Considerations

- **Quotas are an availability control.** Enforcing per-user block and inode limits mitigates local denial-of-service where one account fills a shared filesystem and starves services or logging — aligned with CIS Benchmark guidance to constrain user-writable filesystems.
- **Protect quota database files.** On ext4 the `aquota.user` / `aquota.group` files at the filesystem root govern enforcement; they should be owned by `root` and not writable by users.
- **Combine with hardened mount options.** Pair quotas on `/home` with `nodev,nosuid` (and `noexec` where feasible) so limited storage cannot be abused for privilege escalation staging.

## Troubleshooting

| Symptom | Likely Cause | Check / Fix |
| :-- | :-- | :-- |
| `quotaon: using ... : No such file or directory` | Quota DB not created (ext4) | Run `quotacheck -cugm <mount>` first |
| Limits not enforced after edit | `usrquota`/`grpquota` missing or FS not remounted | Add options to `/etc/fstab`, `mount -o remount` |
| Reported usage looks wrong | Quota accounting drifted | Unmount (or remount RO) and run `quotacheck` |
| XFS quotas do nothing | Mounted without quota option | Remount with `uquota,gquota` or use `xfs_quota -x -c 'enable'` |

## References

- `man 8 quotacheck`, `man 8 quotaon`, `man 8 edquota`, `man 8 repquota`, `man 8 xfs_quota`
- Red Hat Enterprise Linux — Configuring Disk Quotas
- Arch Wiki — Disk quota

## Summary

- Install quota tools → enable in `/etc/fstab` → remount → create quota database (ext4) → enable with `quotaon` → set limits via `edquota` → monitor with `repquota`.
- XFS simplifies this using `xfs_quota`.
- Quotas can be applied per-user and per-group, with **soft** and **hard** limits on **disk blocks and inodes**.

## Related

- [Disk-Management](Disk-Management.md) — overall disk administration
- [Disk-Usage-Check](Disk-Usage-Check.md) — measure consumption against quotas
- [User-and-Group-Management](../Users-Groups-and-Permissions/User-and-Group-Management.md) — quotas are applied per user/group
- [Disk-and-Partition-Management](Disk-and-Partition-Management.md) — filesystems that quotas run on
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
