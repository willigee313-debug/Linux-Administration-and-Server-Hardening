# Exam Preparation

Revision aids for the major Linux administration certifications this course maps to — **spaced-repetition flashcard decks** and **per-exam objective coverage maps**. Use them after working through the module notes and labs to consolidate recall and confirm you have covered every exam objective.

**Course hub:** [Linux Administration & Server Hardening](../Readme.md)

> [!NOTE]
> **How to use these**
> The flashcard decks are written in the Obsidian **spaced-repetition** plugin's inline `Question::Answer` format, so they render as reviewable cards directly in the vault. The objective maps tie each official exam objective to the note(s) that cover it, with a coverage rating, so you can spot and close any weak areas before exam day.

---

## Flashcard Decks

This module contains **8 flashcard decks** grouped by exam domain.

| Deck | Domain |
|------|--------|
| [Flashcards-CLI-Text-and-Files](Flashcards-CLI-Text-and-Files.md) | Command line, text processing, editors, shell environment |
| [Flashcards-Users-Permissions-and-PAM](Flashcards-Users-Permissions-and-PAM.md) | Users, groups, permissions, sudo, PAM, password policy |
| [Flashcards-Storage-and-Packages](Flashcards-Storage-and-Packages.md) | Partitions, LVM, RAID, filesystems, package management |
| [Flashcards-Boot-Services-and-Systemd](Flashcards-Boot-Services-and-Systemd.md) | Boot process, GRUB2, systemd units/targets, rescue mode |
| [Flashcards-Networking](Flashcards-Networking.md) | Interfaces, NetworkManager, routing, VLANs, DNS resolver, SSH |
| [Flashcards-Security-SELinux-and-Firewall](Flashcards-Security-SELinux-and-Firewall.md) | SELinux, firewalld/nftables/iptables, hardening |
| [Flashcards-Virtualization-and-Containers](Flashcards-Virtualization-and-Containers.md) | KVM/libvirt, LXC/LXD, Docker, Podman |
| [Flashcards-Infrastructure-Services](Flashcards-Infrastructure-Services.md) | DNS, DHCP, Apache, Samba, NFS, LDAP, Squid, FTP |

---

## Objective Maps

Each map cross-references the official exam blueprint against this vault's notes, with a per-objective coverage rating (✅ Full · 🟡 Partial · ❌ Gap).

| Map | Exam |
|-----|------|
| [RHCSA-Objective-Mapping](RHCSA-Objective-Mapping.md) | Red Hat Certified System Administrator (EX200) |
| [LFCS-Objective-Mapping](LFCS-Objective-Mapping.md) | Linux Foundation Certified System Administrator |

> [!TIP]
> **Best-fit exams**
> The infrastructure-services depth in this course maps most directly to **RHCSA** and **LFCS**; the CLI, permissions, and networking decks also serve **CompTIA Linux+ (XK0-005)** and **LPIC-1** revision. See the full [Certification Mapping](../Readme.md#certification-mapping) table.

---

## Related

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Practical Labs](../Practical-Labs/Readme.md) — hands-on labs to pair with card review
- [Enterprise Projects](../Enterprise-Projects/Readme.md) — capstone builds that integrate multiple exam domains
