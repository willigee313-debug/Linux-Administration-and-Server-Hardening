# Flashcards — Boot, Services & Systemd

Cards drawn from the **Process, Service and Job Management** module: boot sequence, GRUB2, systemd targets/rescue mode, service management, process/memory tooling, cron, and systemd timers. Study for RHCSA/LFCS/Linux+/LPIC-1.

## Boot Process

What process runs as PID 1 on a modern systemd-based Linux system?::`systemd` (via `/sbin/init` → `/usr/lib/systemd/systemd`)
Which command breaks down boot time into kernel/initrd/userspace phases?::`systemd-analyze`
Which command identifies the longest critical-path chain of units that delayed reaching a target?::`systemd-analyze critical-chain`
What is the purpose of the initramfs during boot?::A temporary RAM-based root filesystem containing the drivers/tools needed to find and mount the real root (LVM/LUKS/RAID/network) before `switch_root`
Which command rebuilds the initramfs for the running kernel on RHEL/CentOS?::`dracut --force /boot/initramfs-$(uname -r).img $(uname -r)`

## GRUB2 Bootloader

Which file holds the human-edited GRUB defaults (timeout, default entry, kernel cmdline) that source `grub.cfg` regeneration?::`/etc/default/grub`
Which command regenerates GRUB2 config on RHEL-family BIOS systems?::`grub2-mkconfig -o /boot/grub2/grub.cfg`
Which command regenerates GRUB2 config on Debian (BIOS or UEFI)?::`update-grub`
Which RHEL-family tool edits kernel command-line args directly in BLS entries without a full `grub2-mkconfig` run?::`grubby` (e.g. `grubby --update-kernel=ALL --args="..."`)
Which command boots a specific GRUB entry once without changing the persistent default?::`grub2-reboot <entry>` (Debian: `grub-reboot`)
Which command sets a GRUB superuser password on RHEL-family systems to require authentication for menu edits?::`grub2-setpassword`

## Systemd Targets & Rescue Mode

Which systemd target is the equivalent of SysV runlevel 5 (networked, graphical login)?::`graphical.target`
Which systemd target mounts only the root filesystem read-only with almost no services started, used to fix a broken `/etc/fstab`?::`emergency.target`
Which command changes the running system's current target immediately (not just at next boot)?::`systemctl isolate <target>`
Which command only sets what boots by default next time, without affecting the current session?::`systemctl set-default <target>`
Which kernel boot parameter drops you into the initramfs before root is mounted, used to reset a forgotten root password on dracut/RHEL systems?::`rd.break`
What must you run after resetting the root password via `rd.break` on an SELinux-enforcing system, before rebooting?::`touch /.autorelabel`

## Service Management (systemctl)

What is the difference between `systemctl restart` and `systemctl reload`?::`restart` stops and starts the daemon (drops connections); `reload` asks it to re-read config without restarting
Which single command both enables a service at boot and starts it immediately?::`systemctl enable --now <unit>`
Which command completely prevents a service from starting even manually, by symlinking its unit to `/dev/null`?::`systemctl mask <unit>`
Which command must be run after editing/adding a unit file so systemd re-reads its definitions?::`systemctl daemon-reload`
Which command lists all failed service units?::`systemctl list-units --type=service --state=failed`

## Process & Memory Management

Which `ps` STAT code indicates a process is in uninterruptible sleep and cannot be killed even with `SIGKILL`?::`D`
What signal number is `SIGKILL`, and can it be caught or ignored by the process?::9; no, it cannot be caught or ignored
Which command brings background job number 2 to the shell foreground?::`fg %2`
Which memory metric should you monitor instead of "free" memory to judge real memory pressure?::`Available` (≈ Free + reclaimable Buff/Cache)
In `vmstat` output, which two columns indicate active swapping (memory pressure)?::`si` (swap-in) and `so` (swap-out)

## Cron Jobs

What are the five fields, in order, of a crontab schedule entry?::minute, hour, day-of-month, month, day-of-week
Which special cron string runs a job once, at system startup?::`@reboot`
Which file, if present, restricts cron use to only the listed users (taking precedence over `cron.deny`)?::`/etc/cron.allow`
Why must scripts placed in `/etc/cron.daily` (and the other cron.* dirs) have no file extension?::`run-parts` skips any filename containing a `.`

## Systemd Timers

What are the two unit types that always work as a pair to define a systemd timer job?::A `.timer` unit (defines *when*) and a `.service` unit (defines *what*)
Which `[Timer]` directive makes a missed scheduled run execute immediately after the next boot?::`Persistent=true`
Which command lists all systemd timers with their next/last elapse times?::`systemctl list-timers --all`

## Related

- [Process, Service & Job Management](../Process-Service-and-Job-Management/Readme.md)
- [Exam Preparation](Readme.md)
- [Linux Administration & Server Hardening](../Readme.md)
