# RHCSA (EX200) Objective Mapping

This map cross-references the Red Hat Certified System Administrator (RHCSA, exam EX200) objective blueprint against the notes in this vault's [Linux Administration & Server Hardening](../Readme.md) course, so you can see coverage at a glance and jump straight to the note that revises a weak objective. It is prep-relevant, not a pass guarantee — hands-on practice on a real RHEL/CentOS Stream/Rocky VM is still required.

> [!NOTE]
> **Blueprint currency**
> Objective wording follows Red Hat's published EX200 exam objectives and may change between exam versions. Always verify against the current exam guide on [access.redhat.com](https://www.redhat.com/en/services/certification/rhcsa) before relying on this map for exam-day scope.

---

## Understand and use essential tools

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Access a shell prompt and issue commands with correct syntax | [Linux-Basic-Commands](../Linux-Basic-Commands/Linux-Basic-Commands.md), [Shells-in-Linux](../Shells-and-Environment/Shells-in-Linux.md) | ✅ Full |
| Use input-output redirection (`>`, `>>`, `\|`, `2>`, etc.) | [Standard-Data-Streams](../Linux-Basic-Commands/Standard-Data-Streams.md), [Multiple-Commands-and-Pipes](../Linux-Basic-Commands/Multiple-Commands-and-Pipes.md) | ✅ Full |
| Use grep and regular expressions to analyze text | [grep-Command](../String-Processing-and-Finding-Files/grep-Command.md), [Searching-Pattern](../String-Processing-and-Finding-Files/Searching-Pattern.md) | ✅ Full |
| Access remote systems using SSH | [SSH-Client-Tools](../SSH-Secure-Shell-Server/SSH-Client-Tools.md), [SSH(Secure-Shell)-Server](../SSH-Secure-Shell-Server/SSH(Secure-Shell)-Server.md) | ✅ Full |
| Log in and switch users in multiple targets | [su-and-sg](../Users-Groups-and-Permissions/su-and-sg.md), [Sudo](../Users-Groups-and-Permissions/Sudo.md) | ✅ Full |
| Archive, compress, unpack, and uncompress files (tar, gzip, bzip2, xz) | [Compress-and-Archive](../Linux-Basic-Commands/Compress-and-Archive.md) | ✅ Full |
| Create and edit text files | [Vi-and-Vim-Editor](../Text-Editors/Vi-and-Vim-Editor.md), [Nano-Editor](../Text-Editors/Nano-Editor.md) | ✅ Full |
| Create, delete, copy, and move files and directories | [Linux-Basic-Commands](../Linux-Basic-Commands/Linux-Basic-Commands.md) | ✅ Full |
| Create hard and soft links | [Linux-Basic-Commands](../Linux-Basic-Commands/Linux-Basic-Commands.md) | ✅ Full |
| List, set, and change standard ugo/rwx permissions | [Linux-Permissions](../Users-Groups-and-Permissions/Linux-Permissions.md), [chmod](../Users-Groups-and-Permissions/chmod.md) | ✅ Full |
| Locate, read, and use system documentation (man, info, `/usr/share/doc`) | [Linux-Basic-Commands](../Linux-Basic-Commands/Linux-Basic-Commands.md) | ✅ Full |

## Create simple shell scripts

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Conditionally execute code (`if`/`test`/`[]`) | [IF-Statement](../Shell-Scripting/IF-Statement.md), [Comparison-Operators](../Shell-Scripting/Comparison-Operators.md) | ✅ Full |
| Use looping constructs (`for`, `while`, `until`) to process file/command-line input | [Shell-Scripting](../Shell-Scripting/Shell-Scripting.md) | 🟡 Partial — loops are demonstrated inline in the main note; no dedicated `for`/`while`/`until` note yet |
| Process script inputs (`$1`, positional params) | [Arguments](../Shell-Scripting/Arguments.md), [getopts](../Shell-Scripting/getopts.md), [Variables](../Shell-Scripting/Variables.md) | ✅ Full |
| Use exit codes and functions | [Exit-status](../Shell-Scripting/Exit-status.md), [Functions](../Shell-Scripting/Functions.md) | ✅ Full |
| Use arrays | [Arrays](../Shell-Scripting/Arrays.md) | ✅ Full |

## Operate running systems

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Boot, reboot, and shut down a system normally | [Linux-Boot-Process](../Process-Service-and-Job-Management/Linux-Boot-Process.md) | ✅ Full |
| Boot systems into different targets manually | [Systemd-Targets-and-Rescue-Mode](../Process-Service-and-Job-Management/Systemd-Targets-and-Rescue-Mode.md) | ✅ Full |
| Interrupt the boot process to gain access to a system | [Reset-Root-Password-and-Protect-GRUB-Boot-Loader](../Security-Firewall-and-Monitoring/Reset-Root-Password-and-Protect-GRUB-Boot-Loader.md), [GRUB2-Bootloader-Configuration](../Process-Service-and-Job-Management/GRUB2-Bootloader-Configuration.md) | ✅ Full |
| Identify CPU/memory intensive processes, adjust priority with `nice`/`renice`, kill processes | [Process-Management-in-Linux](../Process-Service-and-Job-Management/Process-Management-in-Linux.md), [CPU-Performance-Analysis](../Performance-and-Tuning/CPU-Performance-Analysis.md) | ✅ Full |
| Change the current or default systemd target | [Systemd-Targets-and-Rescue-Mode](../Process-Service-and-Job-Management/Systemd-Targets-and-Rescue-Mode.md) | ✅ Full |
| Manage tuning profiles (`tuned`) | — | ❌ Gap — no `tuned`/`tuned-adm` note in this course; add one under Performance-and-Tuning |
| Locate and interpret system log files and the journal | [System-Logging-with-journald](../Monitoring/System-Logging-with-journald.md), [Logging-with-rsyslog](../Monitoring/Logging-with-rsyslog.md) | ✅ Full |
| Preserve system journals (persistent journald storage) | [System-Logging-with-journald](../Monitoring/System-Logging-with-journald.md) | ✅ Full |
| Start, stop, and check the status of network services | [Service-Management-in-Linux](../Process-Service-and-Job-Management/Service-Management-in-Linux.md) | ✅ Full |
| Securely transfer files between systems (scp) | [SSH-Client-Tools](../SSH-Secure-Shell-Server/SSH-Client-Tools.md) | ✅ Full |

## Configure local storage

| Objective | Covering note(s) | Coverage |
|---|---|---|
| List, create, delete MBR and GPT partitions | [Disk-and-Partition-Management](../File-System-and-Disk-Management/Disk-and-Partition-Management.md) | ✅ Full |
| Create and remove physical volumes; assign PVs to volume groups; create/delete logical volumes | [Logical-Volume-Manager(LVM)](../File-System-and-Disk-Management/Logical-Volume-Manager(LVM).md) | ✅ Full |
| Configure systems to mount file systems at boot by UUID or label | [Permanent-Mounting-of-Partitions-in-Linux](../File-System-and-Disk-Management/Permanent-Mounting-of-Partitions-in-Linux.md) | ✅ Full |
| Add new partitions, logical volumes, and swap to a system non-destructively | [Swap-Extend](../File-System-and-Disk-Management/Swap-Extend.md), [Disk-and-Partition-Management](../File-System-and-Disk-Management/Disk-and-Partition-Management.md) | ✅ Full |

## Create and configure file systems

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Create, mount, unmount, and use vfat/ext4/xfs file systems | [Disk-and-Partition-Management](../File-System-and-Disk-Management/Disk-and-Partition-Management.md) | ✅ Full |
| Mount and unmount network file systems using NFS | [NFS-Client-Setup-and-Mounting](../NFS-Server/NFS-Client-Setup-and-Mounting.md), [NFS-Mount-Options](../NFS-Server/NFS-Mount-Options.md) | ✅ Full |
| Extend existing logical volumes | [Logical-Volume-Manager(LVM)](../File-System-and-Disk-Management/Logical-Volume-Manager(LVM).md) | ✅ Full |
| Create and configure set-GID directories for collaboration | [Setgid(Set-Group-ID)](../Users-Groups-and-Permissions/Setgid(Set-Group-ID).md) | ✅ Full |
| Configure disk compression (VDO) | — | ❌ Gap — no VDO note |
| Manage layered storage (Stratis) | — | ❌ Gap — no Stratis note |
| Diagnose and correct file permission problems | [Linux-Permissions](../Users-Groups-and-Permissions/Linux-Permissions.md), [Access-Control-List(ACL)](../Users-Groups-and-Permissions/Access-Control-List(ACL).md) | ✅ Full |

## Deploy, configure, and maintain systems

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Schedule tasks using `at` and `cron` | [Cron-Jobs-in-Linux](../Process-Service-and-Job-Management/Cron-Jobs-in-Linux.md), [Systemd-Timers-in-Linux](../Process-Service-and-Job-Management/Systemd-Timers-in-Linux.md) | 🟡 Partial — cron and systemd timers are covered in depth; no dedicated `at`/`atd` note |
| Start and stop services; configure services to start automatically at boot | [Service-Management-in-Linux](../Process-Service-and-Job-Management/Service-Management-in-Linux.md) | ✅ Full |
| Configure systems to boot into a specific target automatically | [Systemd-Targets-and-Rescue-Mode](../Process-Service-and-Job-Management/Systemd-Targets-and-Rescue-Mode.md) | ✅ Full |
| Configure time service clients (chrony) | [Time-Synchronization-with-chrony](../Network-Configuration/Time-Synchronization-with-chrony.md) | ✅ Full |
| Install and update software packages (repo, local, or filesystem source) | [DNF-Package-Manager](../Package-Management/DNF-Package-Manager.md), [Red-Hat-Package-Manager(RPM)](../Package-Management/Red-Hat-Package-Manager(RPM).md) | ✅ Full |
| Modify the system bootloader | [GRUB2-Bootloader-Configuration](../Process-Service-and-Job-Management/GRUB2-Bootloader-Configuration.md) | ✅ Full |
| Configure IPv4/IPv6 addressing, static routes, hostname resolution, NIC config | [NetworkManager-and-nmcli](../Network-Configuration/NetworkManager-and-nmcli.md), [Static-Routing-and-IP-Forwarding](../Network-Configuration/Static-Routing-and-IP-Forwarding.md) | ✅ Full |

## Manage users and groups

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Create, delete, and modify local user accounts | [User-Management-with-useradd-and-adduser](../Users-Groups-and-Permissions/User-Management-with-useradd-and-adduser.md), [usermod](../Users-Groups-and-Permissions/usermod.md), [userdel](../Users-Groups-and-Permissions/userdel.md) | ✅ Full |
| Change passwords and adjust password aging | [passwd](../Users-Groups-and-Permissions/passwd.md), [Shadow-File-Secure-User-Passwords-File](../Users-Groups-and-Permissions/Shadow-File-Secure-User-Passwords-File.md) | 🟡 Partial — `chage`-specific password-aging workflow is only covered inline in the shadow-file note, not as its own note |
| Create, delete, and modify local groups and group memberships | [groupadd](../Users-Groups-and-Permissions/groupadd.md), [groupmod](../Users-Groups-and-Permissions/groupmod.md), [groupdel](../Users-Groups-and-Permissions/groupdel.md), [gpasswd](../Users-Groups-and-Permissions/gpasswd.md) | ✅ Full |
| Configure superuser access (sudo) | [Sudo](../Users-Groups-and-Permissions/Sudo.md), [Understanding-NOPASSWD-in-sudoers](../Users-Groups-and-Permissions/Understanding-NOPASSWD-in-sudoers.md), [Command-Aliases-in-sudoers](../Users-Groups-and-Permissions/Command-Aliases-in-sudoers.md) | ✅ Full |

## Manage security

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Configure firewall settings using `firewall-cmd`/firewalld | [Firewalld](../Security-Firewall-and-Monitoring/Firewalld.md) | ✅ Full |
| Manage default file permissions | [Linux-Permissions](../Users-Groups-and-Permissions/Linux-Permissions.md) | ✅ Full |
| Configure key-based authentication for SSH | [SSH-Keygen-Usage-and-SSH-Authentication-Setup](../SSH-Secure-Shell-Server/SSH-Keygen-Usage-and-SSH-Authentication-Setup.md), [SSH-Public-and-Private-Key-Configuration](../SSH-Secure-Shell-Server/SSH-Public-and-Private-Key-Configuration.md) | ✅ Full |
| Set enforcing and permissive modes for SELinux | [SELinux-Fundamentals](../Security-Firewall-and-Monitoring/SELinux-Fundamentals.md) | ✅ Full |
| List and identify SELinux file and process context | [SELinux-Contexts-and-File-Labeling](../Security-Firewall-and-Monitoring/SELinux-Contexts-and-File-Labeling.md) | ✅ Full |
| Restore default file contexts | [SELinux-Contexts-and-File-Labeling](../Security-Firewall-and-Monitoring/SELinux-Contexts-and-File-Labeling.md) | ✅ Full |
| Use boolean settings to modify system SELinux settings | [SELinux-Booleans-and-Ports](../Security-Firewall-and-Monitoring/SELinux-Booleans-and-Ports.md) | ✅ Full |
| Diagnose and address routine SELinux policy violations | [SELinux-Troubleshooting](../Security-Firewall-and-Monitoring/SELinux-Troubleshooting.md) | ✅ Full |

## Manage containers

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Find and retrieve container images from a remote registry | [Podman](../Containers/Podman.md), [Docker-Registry](../Containers/Docker-Registry.md) | ✅ Full |
| Inspect container images | [Podman](../Containers/Podman.md) | ✅ Full |
| Perform container management (start, stop, list, remove) using Podman | [Podman](../Containers/Podman.md) | ✅ Full |
| Build a container image from a Containerfile | [Buildah](../Containers/Buildah.md) | ✅ Full |
| Perform basic container management using rootless mode | [Rootless-Containers](../Containers/Rootless-Containers.md) | ✅ Full |
| Run a service as a systemd-managed container (Quadlet / `podman generate systemd`) | [Podman](../Containers/Podman.md) | ✅ Full |
| Configure a container to start automatically as a systemd service | [Podman](../Containers/Podman.md) | ✅ Full |
| Attach persistent storage to a container | [Docker-Volumes-and-Storage](../Containers/Docker-Volumes-and-Storage.md) | 🟡 Partial — this note is Docker-focused, not Podman-specific; `podman volume`/bind-mount syntax differs slightly |

---

## Coverage summary

- ✅ Full: 47
- 🟡 Partial: 5
- ❌ Gap: 3

The three gaps (`tuned` profiles, VDO compression, Stratis layered storage) are all niche "advanced storage/performance" objectives worth light candidates on a live RHEL VM even without a dedicated note — see the `man tuned-adm`, `man vdo`, and `man stratis` pages. The partials are mostly minor (dedicated `at` note, dedicated loop-construct note, dedicated `chage` note, Podman-native persistent-storage note) rather than missing subject matter.

## Related

- [Exam Preparation](Readme.md) — flashcard decks and other objective maps
- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
