# Flashcards — Storage, Filesystems & Packages

Cards drawn from the File-System-and-Disk-Management and Package-Management modules, targeting RHCSA/LFCS/Linux+/LPIC-1 exam objectives on block devices, LVM, RAID, swap, quotas, persistent mounts, and dpkg/apt/rpm/dnf package handling.

## Disk & Partition Identification

Which command lists all block devices and partitions with mount points in a tree format?::`lsblk`
Which command displays a block device's UUID, filesystem TYPE, and LABEL?::`blkid`
Which `lsblk` option combination reports the transport (`sata`/`nvme`/`usb`) and whether a disk is rotational?::`lsblk -d -o NAME,TRAN,ROTA,SIZE,MODEL`
In `fdisk`, which interactive key commits partition-table changes to disk (irreversibly)?::`w`
Which partitioning utility is the GPT-aware counterpart to `fdisk`?::`gdisk`
Which command re-reads/refreshes the kernel's partition table after edits, without a reboot?::`partprobe`
Which command erases stale filesystem signatures from a device before it is repurposed?::`wipefs -a /dev/sdX`
What is the maximum number of primary partitions on an MBR disk (without an extended partition)?::4
How many partitions can a GPT disk support, and how are they classified?::Up to 128, all treated as primary (no extended/logical distinction)

## Filesystems & Persistent Mounts

Which command formats a partition with the ext4 filesystem?::`mkfs.ext4 /dev/sdX1`
Which command formats a partition with XFS, and what is XFS's key resize limitation?::`mkfs.xfs /dev/sdX1`; XFS can only be grown, never shrunk
Which file defines filesystems to be mounted automatically at boot?::`/etc/fstab`
How many fields does each `/etc/fstab` entry contain?::Six — device, mount point, filesystem type, mount options, dump flag, fsck order
Which device identifier type is recommended in `/etc/fstab` over raw device names (e.g. `/dev/sdb1`) for reboot stability?::UUID
Which command tests `/etc/fstab` by mounting all listed filesystems, without rebooting?::`mount -a`
Which three mount options harden a data-only partition against device-node abuse, setuid escalation, and binary execution?::`nodev`, `nosuid`, `noexec`

## LVM & RAID

What is the correct LVM abstraction order from disk to filesystem?::Physical Volume (PV) → Volume Group (VG) → Logical Volume (LV)
Which command initializes a disk or partition as an LVM physical volume?::`pvcreate`
Which command grows a logical volume and resizes its filesystem in a single step?::`lvresize --size <size> --resizefs`
When shrinking an LVM logical volume on ext4, what must be done before `lvreduce`?::Shrink the filesystem first (e.g. `resize2fs` to the target size) — reducing the LV first corrupts live data
Which LVM command relocates extents live off a physical volume onto other PVs in the same volume group?::`pvmove`
Which Linux utility manages software RAID arrays exposed as `/dev/mdN`?::`mdadm`
Which file shows live/real-time RAID array status?::`/proc/mdstat`
What is the minimum number of drives required to build RAID 5?::3
Which RAID level provides disk mirroring with no striping?::RAID 1

## Swap & Quotas

Which command formats a partition or file as swap space?::`mkswap`
Which command activates a swap device or file?::`swapon`
What permission mode should a swap file be set to before enabling it, and why?::`chmod 600` — a world-readable swap file can leak paged-out secrets
Which two `/etc/fstab` mount options enable user and group disk quotas on a filesystem?::`usrquota` and `grpquota`
On ext4, which command builds/updates the quota database files before `quotaon`?::`quotacheck`
What is the difference between a quota's soft limit and hard limit?::Soft limit is a warning threshold that may be exceeded during a grace period; hard limit is an absolute ceiling the kernel refuses to exceed

## Disk Usage

Which command shows free/used space per mounted filesystem in human-readable form?::`df -h`
Why might `df` report a filesystem as full while `du` accounts for far less space used?::A process is still holding a deleted file open; the inode's space isn't released until the last file descriptor closes

## Debian Package Management (dpkg / APT)

Does `dpkg -i` resolve package dependencies automatically?::No — `dpkg` has no dependency awareness; run `apt-get install -f` afterward to fix unmet deps
Which command lists all files installed/owned by a given package?::`dpkg -L package_name`
Which command identifies which installed package owns a given file path?::`dpkg -S /path/to/file`
Which APT command holds a package to prevent it from being upgraded?::`apt-mark hold package_name`
Between `apt` and `apt-get`/`apt-cache`, which is preferred for scripts/automation and why?::`apt-get`/`apt-cache` — their output and behavior are stable across releases, unlike the human-friendly `apt`
Which directory holds APT's local cache of downloaded `.deb` files?::`/var/cache/apt/archives/`

## Red Hat Package Management (RPM / YUM / DNF)

Which command lists all packages installed on an RPM-based system?::`rpm -qa`
Which command identifies which installed package owns a given file?::`rpm -qf /path/to/file`
Which command verifies an installed package's files against the RPM database (integrity check)?::`rpm -V package-name`
Which `rpm` flag lets you inspect an uninstalled `.rpm` file's info before installing it?::`-qp` (e.g. `rpm -qpi package.rpm`)
What does the `--nodeps` flag do when installing/removing with `rpm`, and why is it risky?::Bypasses dependency checks, which can leave software non-functional or break other packages
What replaced YUM as the modern RPM front-end, and what dependency solver does it use?::DNF (Dandified YUM), using the `libsolv` resolver
Which `dnf` command rolls back a specific past transaction by its ID?::`dnf history undo <transaction_id>`
Where is the RPM package database stored, and what command rebuilds it if corrupted?::`/var/lib/rpm`; `rpm --rebuilddb`

## Related

[File System and Disk Management](../File-System-and-Disk-Management/Readme.md)
[Package Management](../Package-Management/Readme.md)
[Exam Preparation](Readme.md)
[Linux Administration & Server Hardening](../Readme.md)
