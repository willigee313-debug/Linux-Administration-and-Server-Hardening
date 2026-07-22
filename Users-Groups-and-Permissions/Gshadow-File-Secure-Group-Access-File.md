# /etc/gshadow – Secure Group Information File

## Overview

The `/etc/gshadow` file stores sensitive group-related information on Linux systems, including group passwords, group administrators, and member lists. It is the secure counterpart to `/etc/group` and protects group authentication data from unauthorized access.

Unlike `/etc/group`, which is readable by all users, `/etc/gshadow` is readable only by root. This mirrors the `/etc/passwd` → `/etc/shadow` split for user accounts.

## Concepts

### Purpose of `/etc/gshadow`

The file is used to:

- Store encrypted group passwords.
- Define group administrators.
- Maintain secure group membership information.
- Support the `newgrp` command for temporary group changes.
- Prevent exposure of sensitive group authentication data.

## Architecture

### Format of `/etc/gshadow`

Each line represents a group and uses a colon-separated format:

```text
group_name:encrypted_password:administrators:members
```

| Field No. | Field name | Description |
| :-- | :-- | :-- |
| 1 | `group_name` | Name of the group |
| 2 | `encrypted_password` | Encrypted group password or status indicator |
| 3 | `administrators` | Comma-separated list of group administrators |
| 4 | `members` | Comma-separated list of group members |

### Example Entries

```text
root:::
daemon:::
sshd:!::
apache:!::
developers:!!:john:sarah,mike
```

Breaking down the `developers` entry:

| Field | Value | Description |
| :-- | :-- | :-- |
| Group name | `developers` | Name of the group |
| Password | `!!` | Password is locked or not set |
| Administrators | `john` | Group administrator |
| Members | `sarah,mike` | Group members |

### Understanding the Password Field

The second field contains password information or a special symbol:

| Value | Meaning |
| :-- | :-- |
| `!` | Group password disabled |
| `!!` | Password not initialized or locked |
| `*` | Authentication disabled |
| Encrypted hash | Valid encrypted group password |
| Empty field | No password configured |

Locked group:

```text
developers:!::john,mike
```

Group with an encrypted password:

```text
project:$6$abc123xyz...::john,mike
```

> [!NOTE]
> Most modern Linux systems do not use group passwords, so `!` or `!!` are what you will typically see in this field.

### Group Administrators

The third field lists users who can administer the group using `gpasswd`.

```text
developers:!:john:sarah,mike
```

Here, user `john` can:

- Add members.
- Remove members.
- Change the group password.
- Manage the group without editing the file directly.

View administrators:

```bash
getent gshadow developers
```

### Group Members

The fourth field lists supplementary group members.

```text
developers:!:john:sarah,mike,david
```

Members: `sarah`, `mike`, `david`. These users gain the permissions associated with the group.

### Relationship with `/etc/group`

The two files describe the same groups from different angles and must stay synchronized.

`/etc/group` — group name, GID, and member list:

```text
developers:x:1001:sarah,mike,david
```

`/etc/gshadow` — secure password data, administrators, and member list:

```text
developers:!:john:sarah,mike,david
```

## Commands

### Viewing `/etc/gshadow`

Display contents (root only):

```bash
sudo cat /etc/gshadow
```

View a specific group:

```bash
sudo getent gshadow developers
```

Search for a group:

```bash
sudo grep "^developers:" /etc/gshadow
```

### Managing Group Membership

Add a user to a group:

```bash
gpasswd -a username groupname
```

```bash
gpasswd -a john developers
```

```text
Adding user john to group developers
```

Remove a user from a group:

```bash
gpasswd -d john developers
```

```text
Removing user john from group developers
```

### Managing Group Administrators

Assign a group administrator:

```bash
gpasswd -A john developers
```

Assign multiple administrators:

```bash
gpasswd -A john,mike developers
```

Verify:

```bash
sudo getent gshadow developers
```

### Managing Group Passwords

Set a group password (you will be prompted to enter one):

```bash
gpasswd developers
```

Remove a group password:

```bash
gpasswd -r developers
```

Restrict group access (disables access via group passwords):

```bash
gpasswd -R developers
```

### Using `newgrp`

The `newgrp` command switches the user's current active group:

```bash
newgrp developers
```

If the user is not already a member, a group password may be requested.

Verify the current group:

```bash
id
```

## Configuration

### Safe Editing

Manual editing is discouraged because `/etc/group` and `/etc/gshadow` must remain synchronized.

Edit `/etc/gshadow` safely:

```bash
vigr -s
```

Edit `/etc/group`:

```bash
vigr
```

> [!TIP]
> `vigr` / `vigr -s` lock the files during editing and perform consistency checks. Prefer `gpasswd` for routine membership and administrator changes so both files stay in sync automatically.

### File Permissions

Check the permissions:

```bash
ls -l /etc/gshadow
```

```text
-r-------- 1 root root 850 Jun 10 12:00 /etc/gshadow
```

Typical ownership and mode:

```text
Owner: root
Group: root
Permissions: 0400
```

Only root should have read access.

### Consistency Checking

Verify group database integrity:

```bash
grpck
```

Check both files explicitly:

```bash
grpck /etc/group /etc/gshadow
```

```text
group developers: no problems
```

## Reference

### Related Files

| File | Purpose |
| :-- | :-- |
| `/etc/group` | Group names, GIDs, and member lists |
| `/etc/gshadow` | Secure group passwords, administrators, and members |
| `/etc/passwd` | User account information |
| `/etc/shadow` | User password hashes and password-aging policies |

### Command Summary

| Task | Command |
| :-- | :-- |
| View group information | `getent group groupname` |
| View secure group information | `sudo getent gshadow groupname` |
| Add user to group | `gpasswd -a user group` |
| Remove user from group | `gpasswd -d user group` |
| Assign group administrator | `gpasswd -A user group` |
| Set group password | `gpasswd groupname` |
| Remove group password | `gpasswd -r groupname` |
| Restrict group password usage | `gpasswd -R groupname` |
| Switch to group | `newgrp groupname` |
| Edit securely | `vigr -s` |
| Verify integrity | `grpck` |

## Security Considerations

- Never make `/etc/gshadow` world-readable — mode `0400`, owner `root`, keeps password hashes and administrator data private.
- Use `gpasswd` instead of manual editing whenever possible so `/etc/group` and `/etc/gshadow` stay synchronized.
- Avoid using group passwords unless specifically required; a shared group password is a poor secret and is hard to rotate. Prefer explicit membership.
- Regularly audit group memberships and administrators, especially for privileged groups.
- Run `grpck` after any manual modification to catch inconsistencies between the two files.

## Troubleshooting

| Symptom | Likely cause | Fix |
| :-- | :-- | :-- |
| `newgrp` prompts for a password unexpectedly | User not a member of the target group | Add the user with `gpasswd -a user group`, or check the member list |
| `grpck` reports inconsistencies | `/etc/group` and `/etc/gshadow` diverged | Reconcile with `gpasswd`/`vigr`; re-run `grpck` |
| Non-root user cannot read `/etc/gshadow` | Correct behavior — file is mode `0400` | Use `sudo getent gshadow groupname` instead |
| Administrator cannot manage a group | User not listed in the administrators field | Assign with `gpasswd -A user group` |

## Related

- [Group-File-Linux-Group-Account-File](Group-File-Linux-Group-Account-File.md) — the world-readable group file this one secures
- [Passwd-File-Linux-User-Account-File](Passwd-File-Linux-User-Account-File.md) — user account file (`/etc/passwd`)
- [Shadow-File-Secure-User-Passwords-File](Shadow-File-Secure-User-Passwords-File.md) — user-side equivalent (`/etc/shadow`)
- [gpasswd](gpasswd.md) — administer group members, admins, and passwords
- [su-and-sg](su-and-sg.md) — switch user/group identity, related to `newgrp`
- [User-and-Group-Management](User-and-Group-Management.md) — broader user and group administration workflow
- [Linux Administration & Server Hardening](../Readme.md) — course hub.
