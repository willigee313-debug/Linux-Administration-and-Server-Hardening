# LFCS Objective Mapping

Maps the official **Linux Foundation Certified System Administrator (LFCS)** exam domains to the notes in this vault's [Linux Administration & Server Hardening](../Readme.md) course, so you can see coverage at a glance and jump straight to the right note to revise. This is exam-prep support, not a guarantee of passing.

> [!NOTE]
> **Objective wording**
> Domain and objective wording below follows the publicly documented LFCS blueprint structure (Essential Commands, Operations Deployment, Networking, Storage, Users and Groups, Service Configuration, Security) as paraphrased task areas, not a verbatim reproduction of the Linux Foundation's exam guide. The blueprint is revised periodically — verify current weighting and wording against the official exam guide before scheduling.

---

## Essential Commands

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Navigate the filesystem; manage files and directories | [Linux-Basic-Commands](../Linux-Basic-Commands/Linux-Basic-Commands.md) | ✅ Full |
| Archive, compress, and unpack files | [Compress-and-Archive](../Linux-Basic-Commands/Compress-and-Archive.md) | ✅ Full |
| Use I/O redirection, pipes, and multiple commands | [Standard-Data-Streams](../Linux-Basic-Commands/Standard-Data-Streams.md), [Multiple-Commands-and-Pipes](../Linux-Basic-Commands/Multiple-Commands-and-Pipes.md) | ✅ Full |
| Search files and content with grep/find | [grep-Command](../String-Processing-and-Finding-Files/grep-Command.md), [Find-Command](../String-Processing-and-Finding-Files/Find-Command.md) | ✅ Full |
| Process text streams with sed/awk/cut | [sed](../String-Processing-and-Finding-Files/sed.md), [awk-Command](../String-Processing-and-Finding-Files/awk-Command.md), [cut-Command](../String-Processing-and-Finding-Files/cut-Command.md) | ✅ Full |
| Edit text files with vi/vim | [Vi-and-Vim-Editor](../Text-Editors/Vi-and-Vim-Editor.md), [Vim-Command](../Text-Editors/Vim-Command.md) | ✅ Full |
| Use terminal multiplexers for persistent sessions | [Screen-Command](../Linux-Basic-Commands/Screen-Command.md) | 🟡 Partial — `screen` only, no `tmux` note in this course |
| Write and execute simple shell scripts | [Shell-Scripting](../Shell-Scripting/Shell-Scripting.md) | ✅ Full |
| Use shell/environment variables | [Linux-Environment-Variables](../Shells-and-Environment/Linux-Environment-Variables.md) | ✅ Full |
| Compare and diff text files | [File-Comparison-Tools-in-Linux](../String-Processing-and-Finding-Files/File-Comparison-Tools-in-Linux.md) | 🟡 Partial — covers `diff`/`cmp`-style tools, not a dedicated `diff` deep-dive |

## Operations Deployment

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Understand the boot sequence and GRUB2 | [Linux-Boot-Process](../Process-Service-and-Job-Management/Linux-Boot-Process.md), [GRUB2-Bootloader-Configuration](../Process-Service-and-Job-Management/GRUB2-Bootloader-Configuration.md) | ✅ Full |
| Boot to alternate systemd targets / rescue mode | [Systemd-Targets-and-Rescue-Mode](../Process-Service-and-Job-Management/Systemd-Targets-and-Rescue-Mode.md) | ✅ Full |
| Diagnose and manage processes | [Process-Management-in-Linux](../Process-Service-and-Job-Management/Process-Management-in-Linux.md) | ✅ Full |
| Create and manage systemd unit services | [Service-Management-in-Linux](../Process-Service-and-Job-Management/Service-Management-in-Linux.md) | ✅ Full |
| Schedule recurring/one-off jobs (cron, at, timers) | [Cron-Jobs-in-Linux](../Process-Service-and-Job-Management/Cron-Jobs-in-Linux.md), [Systemd-Timers-in-Linux](../Process-Service-and-Job-Management/Systemd-Timers-in-Linux.md) | ✅ Full |
| Install and manage software packages | [DNF-Package-Manager](../Package-Management/DNF-Package-Manager.md), [Advanced-Package-Tool(APT)](../Package-Management/Advanced-Package-Tool(APT).md) | ✅ Full |
| Deploy and manage containers | [Introduction-to-Containers](../Containers/Introduction-to-Containers.md), [Docker-Containers-Lifecycle](../Containers/Docker-Containers-Lifecycle.md), [Podman](../Containers/Podman.md) | ✅ Full |
| Deploy and manage basic virtual machines | [Introduction-to-Virtualization](../Virtualization/Introduction-to-Virtualization.md), [KVM-Kernel-Virtual-Machine](../Virtualization/KVM-Kernel-Virtual-Machine.md), [libvirt-and-virsh](../Virtualization/libvirt-and-virsh.md) | ✅ Full |
| Basic version control (git) | — | ❌ Gap — no git note exists in this course; not covered anywhere in the vault under this folder |
| Use system logs to troubleshoot | [System-Logging-with-journald](../Monitoring/System-Logging-with-journald.md), [Logging-with-rsyslog](../Monitoring/Logging-with-rsyslog.md) | ✅ Full |

## Networking

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Configure network interfaces and addressing | [ifconfig-and-ip](../Network-Configuration/ifconfig-and-ip.md), [NetworkManager-and-nmcli](../Network-Configuration/NetworkManager-and-nmcli.md) | ✅ Full |
| Configure static routing / IP forwarding | [Static-Routing-and-IP-Forwarding](../Network-Configuration/Static-Routing-and-IP-Forwarding.md) | ✅ Full |
| Perform name resolution / use DNS client tools | [Dns-Client-Tools](../Domain-Name-System-DNS/Dns-Client-Tools.md), [dig](../Domain-Name-System-DNS/dig.md), [nslookup](../Domain-Name-System-DNS/nslookup.md) | ✅ Full |
| Configure host-based firewall rules | [Firewalld](../Security-Firewall-and-Monitoring/Firewalld.md), [nftables-Configuration-and-Management](../Security-Firewall-and-Monitoring/nftables-Configuration-and-Management.md) | ✅ Full |
| Synchronize system time (NTP/chrony) | [Time-Synchronization-with-chrony](../Network-Configuration/Time-Synchronization-with-chrony.md) | ✅ Full |
| Configure and secure SSH remote access | [SSH(Secure-Shell)-Server](../SSH-Secure-Shell-Server/SSH(Secure-Shell)-Server.md), [SSH-Public-and-Private-Key-Configuration](../SSH-Secure-Shell-Server/SSH-Public-and-Private-Key-Configuration.md) | ✅ Full |
| Diagnose network connectivity and sockets | [Network-Diagnostics-Commands](../Network-Configuration/Network-Diagnostics-Commands.md), [Network-Monitoring-netstat-and-ss-Commands](../Network-Configuration/Network-Monitoring-netstat-and-ss-Commands.md) | ✅ Full |
| Configure VLANs and network bonding | [VLANs-and-Network-Bonding](../Network-Configuration/VLANs-and-Network-Bonding.md) | ✅ Full |

## Storage

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Partition disks (fdisk/parted) | [Disk-and-Partition-Management](../File-System-and-Disk-Management/Disk-and-Partition-Management.md) | ✅ Full |
| Create and manage LVM (PV/VG/LV) | [Logical-Volume-Manager(LVM)](../File-System-and-Disk-Management/Logical-Volume-Manager(LVM).md) | ✅ Full |
| Create and manage filesystems | [Disk-Management](../File-System-and-Disk-Management/Disk-Management.md) | ✅ Full |
| Mount/unmount filesystems and configure fstab | [Permanent-Mounting-of-Partitions-in-Linux](../File-System-and-Disk-Management/Permanent-Mounting-of-Partitions-in-Linux.md) | ✅ Full |
| Configure and extend swap | [Swap-Extend](../File-System-and-Disk-Management/Swap-Extend.md) | ✅ Full |
| Configure software RAID | [RAID-in-Linux](../File-System-and-Disk-Management/RAID-in-Linux.md) | ✅ Full |
| Manage disk quotas | [Disk-Quota-Management-in-Linux](../File-System-and-Disk-Management/Disk-Quota-Management-in-Linux.md) | ✅ Full |
| Monitor disk/filesystem usage | [Disk-Usage-Check](../File-System-and-Disk-Management/Disk-Usage-Check.md) | ✅ Full |
| Manage file ACLs beyond standard permissions | [Access-Control-List(ACL)](../Users-Groups-and-Permissions/Access-Control-List(ACL).md) | ✅ Full |

## Users and Groups

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Create, modify, and delete user accounts | [User-and-Group-Management](../Users-Groups-and-Permissions/User-and-Group-Management.md), [User-Management-with-useradd-and-adduser](../Users-Groups-and-Permissions/User-Management-with-useradd-and-adduser.md) | ✅ Full |
| Create and manage groups | [groupadd](../Users-Groups-and-Permissions/groupadd.md), [groupmod](../Users-Groups-and-Permissions/groupmod.md) | ✅ Full |
| Configure sudo access and command aliases | [Sudo](../Users-Groups-and-Permissions/Sudo.md), [Command-Aliases-in-sudoers](../Users-Groups-and-Permissions/Command-Aliases-in-sudoers.md) | ✅ Full |
| Manage password aging and quality policy | [Password-Policy-with-pam_pwquality](../Users-Groups-and-Permissions/Password-Policy-with-pam_pwquality.md), [passwd](../Users-Groups-and-Permissions/passwd.md) | ✅ Full |
| Configure PAM stacks | [PAM-Pluggable-Authentication-Modules](../Users-Groups-and-Permissions/PAM-Pluggable-Authentication-Modules.md) | ✅ Full |
| Lock accounts after failed logins | [Account-Lockout-with-pam_faillock](../Users-Groups-and-Permissions/Account-Lockout-with-pam_faillock.md) | ✅ Full |
| Recover a lost root password | [Reset-the-Root-Password-in-CentOS-Stream-10](../Users-Groups-and-Permissions/Reset-the-Root-Password-in-CentOS-Stream-10.md) | ✅ Full |

## Service Configuration

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Deploy and configure a web server (Apache) | [Apache-Web-Server-Setup-and-Configuration](../Apache-Web-Server/Apache-Web-Server-Setup-and-Configuration.md) | ✅ Full |
| Deploy and configure an authoritative DNS server | [DNS-Server-Types](../Domain-Name-System-DNS/DNS-Server-Types.md), [Master-Nameserver](../Domain-Name-System-DNS/Master-Nameserver.md) | ✅ Full |
| Deploy and configure a DHCP server | [DHCP-Server](../Dynamic-Host-Configuration-Protocol-DHCP/DHCP-Server.md) | ✅ Full |
| Deploy and configure a database service | [PHP-and-Mysql-Installation-and-Configuration](../Apache-Web-Server/PHP-and-Mysql-Installation-and-Configuration.md) | 🟡 Partial — install/wiring only, no dedicated MariaDB/MySQL administration note |
| Configure network file sharing (NFS/Samba) | [Network-File-System-(NFS)-Server](../NFS-Server/Network-File-System-(NFS)-Server.md), [Samba-SMB-CIFS-Server](../Samba-SMB-CIFS-Server/Samba-SMB-CIFS-Server.md) | ✅ Full |
| Deploy and configure a mail service | — | ❌ Gap — no mail-server note in this course |
| Understand basic container orchestration | [Kubernetes-Basics](../Containers/Kubernetes-Basics.md) | 🟡 Partial — conceptual intro only, no cluster-admin depth |

## Security

| Objective | Covering note(s) | Coverage |
|---|---|---|
| Manage file permissions and ownership | [Linux-Permissions](../Users-Groups-and-Permissions/Linux-Permissions.md), [chmod](../Users-Groups-and-Permissions/chmod.md), [chown](../Users-Groups-and-Permissions/chown.md) | ✅ Full |
| Apply special permissions (SUID/SGID/sticky bit) | [Setuid(Set-User-ID)](../Users-Groups-and-Permissions/Setuid(Set-User-ID).md), [Setgid(Set-Group-ID)](../Users-Groups-and-Permissions/Setgid(Set-Group-ID).md), [Sticky-Bit](../Users-Groups-and-Permissions/Sticky-Bit.md) | ✅ Full |
| Configure and troubleshoot SELinux | [SELinux-Fundamentals](../Security-Firewall-and-Monitoring/SELinux-Fundamentals.md), [SELinux-Contexts-and-File-Labeling](../Security-Firewall-and-Monitoring/SELinux-Contexts-and-File-Labeling.md), [SELinux-Troubleshooting](../Security-Firewall-and-Monitoring/SELinux-Troubleshooting.md) | ✅ Full |
| Configure and troubleshoot AppArmor | — | ❌ Gap — no AppArmor note in this course (SELinux is the only MAC framework covered) |
| Configure firewalld/nftables rulesets | [Firewalld](../Security-Firewall-and-Monitoring/Firewalld.md), [nftables-Configuration-and-Management](../Security-Firewall-and-Monitoring/nftables-Configuration-and-Management.md) | ✅ Full |
| Restrict privilege escalation via sudo | [Sudo](../Users-Groups-and-Permissions/Sudo.md), [Understanding-NOPASSWD-in-sudoers](../Users-Groups-and-Permissions/Understanding-NOPASSWD-in-sudoers.md) | ✅ Full |
| Harden SSH access | [Prevent-Root-Login-via-SSH](../SSH-Secure-Shell-Server/Prevent-Root-Login-via-SSH.md), [Change-Default-SSH-Port](../SSH-Secure-Shell-Server/Change-Default-SSH-Port.md) | ✅ Full |
| Audit system activity | [Auditing-with-auditd](../Monitoring/Auditing-with-auditd.md) | ✅ Full |
| Scan for malware | [ClamAV-Antivirus](../Security-Firewall-and-Monitoring/ClamAV-Antivirus.md) | 🟡 Partial — single tool covered, no broader host-AV comparison |
| Secure the bootloader / recover root safely | [Reset-Root-Password-and-Protect-GRUB-Boot-Loader](../Security-Firewall-and-Monitoring/Reset-Root-Password-and-Protect-GRUB-Boot-Loader.md) | ✅ Full |

---

## Coverage summary

| Domain | ✅ Full | 🟡 Partial | ❌ Gap | Objectives |
|---|---|---|---|---|
| Essential Commands | 8 | 2 | 0 | 10 |
| Operations Deployment | 9 | 0 | 1 | 10 |
| Networking | 8 | 0 | 0 | 8 |
| Storage | 9 | 0 | 0 | 9 |
| Users and Groups | 7 | 0 | 0 | 7 |
| Service Configuration | 4 | 2 | 1 | 7 |
| Security | 8 | 1 | 1 | 10 |
| **Total** | **53** | **5** | **3** | **61** |

> [!TIP]
> **Closing the gaps**
> The three gaps (git basics, mail service, AppArmor) are not covered anywhere in this course folder. If your target exam version weights any of these heavily, supplement from outside this vault before relying on it as your sole revision source.

## Related

- [Exam Preparation](Readme.md) — flashcard decks and the sibling [RHCSA-Objective-Mapping](RHCSA-Objective-Mapping.md)
- [Linux Administration & Server Hardening](../Readme.md) — course hub and full module index
