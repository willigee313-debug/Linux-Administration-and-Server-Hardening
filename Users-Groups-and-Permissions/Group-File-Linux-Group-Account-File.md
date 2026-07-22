# /etc/group – Linux Group Account File

## Overview

The `/etc/group` file stores information about **groups** on a Linux system. Groups provide a way to manage permissions and access controls for multiple users collectively, rather than assigning rights to each user one at a time. Together with `/etc/passwd` (user accounts) and `/etc/gshadow` (secure group data), it forms the core of the local group database.

> [!NOTE]
> `/etc/group` is world-readable by design — any user can list group names, GIDs, and membership. Sensitive data such as group password hashes lives in `/etc/gshadow`, which is readable only by root.

## Concepts

### Purpose of Groups

Groups allow administrators to:

- Assign permissions to multiple users at once.
- Control access to files, directories, devices, and services.
- Separate administrative and non-administrative users.
- Simplify user management in multi-user environments.

### Primary vs Supplementary Groups

Every user has exactly one **primary group** and may belong to any number of **supplementary groups**.

| Type | Where defined | Notes |
| :-- | :-- | :-- |
| Primary group | 4th field of the user's `/etc/passwd` entry | Owns files the user creates by default |
| Supplementary groups | Member list (4th field) in `/etc/group` | Additional access; a user can belong to many |

Primary group — defined in `/etc/passwd`:

```text
john:x:1000:1000:John Doe:/home/john:/bin/bash
```

The fourth field (`1000`) is the GID of John's primary group. Verify with:

```bash
id john
```

```text
uid=1000(john) gid=1000(john) groups=1000(john),10(wheel),1001(developers)
```

Supplementary groups — additional entries in `/etc/group`:

```text
developers:x:1001:john
wheel:x:10:john
```

## Architecture

### Format of `/etc/group`

Each line represents a single group and uses a colon-separated format:

```text
group_name:password:GID:user_list
```

| Field No. | Field name | Description |
| :-- | :-- | :-- |
| 1 | `group_name` | Name of the group |
| 2 | `password` | Group password field, usually `x` or empty |
| 3 | `GID` | Group ID (unique numeric identifier) |
| 4 | `user_list` | Comma-separated list of supplementary group members |

### Example

```text
root:x:0:
daemon:x:2:
sshd:x:74:
apache:x:48:
developers:x:1001:john,sarah,mike
```

Breaking down the `developers` entry:

| Field | Value | Description |
| :-- | :-- | :-- |
| Group name | `developers` | Name of the group |
| Password | `x` | Placeholder for group password |
| GID | `1001` | Unique group identifier |
| Members | `john,sarah,mike` | Users belonging to the group |

## Commands

### Viewing Group Information

Display all groups:

```bash
cat /etc/group
```

Search for a specific group:

```bash
grep "^developers:" /etc/group
```

or query the name-service database (also covers LDAP/SSSD-backed groups):

```bash
getent group developers
```

Display the current user's groups:

```bash
groups
```

Display groups for a specific user:

```bash
groups username
```

Display detailed user and group information:

```bash
id username
```

```bash
id john
```

```text
uid=1000(john) gid=1000(john) groups=1000(john),10(wheel),1001(developers)
```

### Group Administration Commands

Create a new group:

```bash
groupadd devteam
```

Create a group with a specific GID:

```bash
groupadd -g 2000 devteam
```

Add a user to a group (the `-a` in `-aG` *appends*; omitting it replaces all supplementary groups):

```bash
usermod -aG devteam username
```

```bash
usermod -aG devteam john
```

Verify:

```bash
groups john
```

Remove a user from a group:

```bash
gpasswd -d john devteam
```

Change a group name:

```bash
groupmod -n developers devteam
```

Change a group ID:

```bash
groupmod -g 3000 developers
```

Delete a group:

```bash
groupdel developers
```

## Configuration

### Editing `/etc/group` Safely

Although the group-management commands above are preferred, the file can be edited manually. Always back it up first:

```bash
cp /etc/group /etc/group.bak
```

Edit with the locking editor:

```bash
vipw -g
```

or:

```bash
sudo vim /etc/group
```

> [!TIP]
> `vipw -g` is preferred because it locks the file during editing and helps prevent corruption from concurrent changes. Use the group-management commands (`groupadd`, `groupmod`, `usermod`, `gpasswd`) whenever possible so `/etc/group` and `/etc/gshadow` stay synchronized.

### Group Passwords

Linux supports group passwords through the second field, but this feature is rarely used today.

```text
project:$6$hashedpassword:2000:
```

Modern systems typically leave a placeholder instead:

```text
project:x:2000:
```

Actual group password hashes, when used, are stored in `/etc/gshadow` — not in `/etc/group`.

## Reference

### Common System Groups

| Group | Purpose |
| :-- | :-- |
| `root` | Administrative group |
| `wheel` | Sudo access on many distributions |
| `sudo` | Administrative privileges on Debian/Ubuntu |
| `audio` | Audio device access |
| `video` | Graphics and video device access |
| `disk` | Raw disk access |
| `cdrom` | Optical drive access |
| `lp` | Printing services |
| `sshd` | SSH daemon operations |
| `apache` | Apache web server processes |
| `mysql` | MySQL/MariaDB service account |
| `docker` | Docker daemon access |
| `kvm` | KVM virtualization access |
| `libvirt` | Virtual machine management |
| `postfix` | Mail server operations |

### Related Files

| File | Purpose |
| :-- | :-- |
| `/etc/group` | Group names, GIDs, and supplementary member lists |
| `/etc/gshadow` | Secure group passwords and group administrators |
| `/etc/passwd` | User account information, references each user's primary group |

Example `/etc/gshadow` entry (locked, no members):

```text
developers:!::
```

Example `/etc/passwd` entry referencing a primary group:

```text
john:x:1000:1000:John Doe:/home/john:/bin/bash
```

### Command Summary

| Task | Command |
| :-- | :-- |
| View all groups | `cat /etc/group` |
| View a specific group | `getent group groupname` |
| View a user's groups | `groups username` |
| Show detailed membership | `id username` |
| Create a group | `groupadd groupname` |
| Add user to group | `usermod -aG groupname username` |
| Remove user from group | `gpasswd -d username groupname` |
| Rename a group | `groupmod -n newgroup oldgroup` |
| Change GID | `groupmod -g GID groupname` |
| Delete a group | `groupdel groupname` |
| Edit group file safely | `vipw -g` |

## Security Considerations

- Grant access through **groups**, not by assigning permissions directly to individual users — it is easier to audit and revoke.
- Avoid adding users unnecessarily to privileged groups such as `wheel`, `sudo`, `docker`, or `disk`. Membership in `docker` or `disk` is effectively root-equivalent.
- Regularly audit group memberships:

```bash
getent group
```

- Review high-value privileged groups specifically:

```bash
getent group wheel
```

```bash
getent group sudo
```

```bash
getent group docker
```

- Apply least-privilege principles when assigning users to groups (aligned with CIS Benchmark guidance for account and access control).

## Troubleshooting

| Symptom | Likely cause | Fix |
| :-- | :-- | :-- |
| User's new group not active | Group membership loaded at login | Re-login, or run `newgrp groupname` for the current shell |
| `usermod -G` removed other groups | Used `-G` without `-a` | Always use `usermod -aG` to append supplementary groups |
| `/etc/group` and `/etc/gshadow` out of sync | Manual edit of only one file | Run `grpck` and prefer `gpasswd`/`groupmod` over hand-editing |
| Duplicate GID warnings | Two groups share a GID | Assign a unique GID with `groupmod -g` |

## Related

- [Gshadow-File-Secure-Group-Access-File](Gshadow-File-Secure-Group-Access-File.md) — secure counterpart storing group password hashes and administrators
- [Passwd-File-Linux-User-Account-File](Passwd-File-Linux-User-Account-File.md) — user account file that references each user's primary group
- [groupadd](groupadd.md) — create groups
- [groupmod](groupmod.md) — modify group name or GID
- [groupdel](groupdel.md) — delete groups
- [gpasswd](gpasswd.md) — administer group membership, admins, and passwords
- [usermod](usermod.md) — add users to supplementary groups
- [User-and-Group-Management](User-and-Group-Management.md) — broader user and group administration workflow
- [Linux Administration & Server Hardening](../Readme.md) — course hub.
