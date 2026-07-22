# deluser

## Overview

`deluser` is the Debian/Ubuntu-family utility for removing user accounts and stripping users from supplementary groups. It is a Perl wrapper around the low-level `userdel` and `groupdel` tools, driven by the policy file `/etc/deluser.conf`. Compared with `userdel`, it offers a safer, more explicit interface: sane defaults, clearer flags for home-directory and file removal, and optional pre/post scripts.

> [!NOTE]
> `deluser` ships only on Debian-derived distributions (Debian, Ubuntu, Kali, Mint). On RHEL, Fedora, CentOS Stream, SUSE, and Arch use `userdel` directly — see [userdel](userdel.md).

## Concepts

Removing an account is more than deleting a line in `/etc/passwd`. A complete removal touches several places:

| Artifact | Location | Removed by |
| --- | --- | --- |
| Account record | `/etc/passwd` | Always |
| Password hash | `/etc/shadow` | Always |
| Group memberships | `/etc/group`, `/etc/gshadow` | Always |
| Home directory | `/home/<user>` | `--remove-home` |
| Files elsewhere on disk | Entire filesystem | `--remove-all-files` |
| Primary group | `/etc/group` | Auto if same-name group is empty |

> [!WARNING]
> Files owned by a deleted user that are **not** removed become "orphaned" — owned by a bare numeric UID. If that UID is later recycled for a new account, the new user silently inherits ownership of those files. Decide deliberately whether to delete, archive, or reassign a departing user's files.

## Options

| Option | Purpose |
| --- | --- |
| `--remove-home` | Delete the user's home directory and mail spool. |
| `--remove-all-files` | Delete every file on the system owned by the user (implies `--remove-home`). |
| `--backup` | Back up files to a tarball before deletion. |
| `--backup-to <dir>` | Directory to write the backup into. |
| `--force` | Proceed even when the account appears to be in use. |
| `<user> <group>` | Two positional arguments: remove the user from that supplementary group only. |
| `--help` | Show usage information. |

## Commands

- Remove user `u15` without deleting their home directory.

```bash
deluser u15
```

- Remove user `u1` and delete their home directory.

```bash
deluser --remove-home u1
```

- Remove user `u16`, delete their home directory, and remove all files on the system owned by that user.

```bash
deluser --remove-home --remove-all-files u16
```

- Remove user `u17` from the supplementary group `developers`.

```bash
deluser u17 developers
```

- Forcefully remove user `u18`, even if the account is currently in use.

```bash
deluser --force u18
```

- Show available options and usage information.

```bash
deluser --help
```

## Examples

Decommission a departing employee while preserving their data for handover — back up first, then remove the account and its home directory:

```bash
# Archive the home directory to /backup before removal
deluser --backup --backup-to /backup --remove-home jsmith
```

Remove a service account and everything it owned across the filesystem:

```bash
deluser --remove-all-files svc_deploy
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: Terminal showing deluser removing an account, the informational messages about the home directory being deleted, and a follow-up id command reporting no such user_

## Best Practices

> [!TIP]
> - Lock and disable the account before deletion (`usermod -L -e 1 <user>`), confirm no active sessions with `who`/`loginctl`, then delete. This prevents the user from logging back in during the offboarding window.
> - Prefer `--backup` for human accounts so data can be recovered or handed over.
> - Kill lingering processes owned by the user before `--force`; a forced removal does not clean up running processes.

## Security Considerations

- **Orphaned files** (see [Setuid(Set-User-ID)](Setuid(Set-User-ID).md) risks): audit for files owned by the removed UID with `find / -uid <old_uid> 2>/dev/null` before recycling the UID.
- **Cron and at jobs**: `deluser` does not always remove a user's scheduled jobs. Check `/var/spool/cron/crontabs/<user>` and the `at` queue.
- **Group cleanup**: removing the last member of a project group leaves an empty group behind; delete it with [groupdel](groupdel.md) if it is no longer needed.
- Aligns with CIS Benchmark guidance on prompt removal of unused and terminated accounts to reduce the attack surface.

## Troubleshooting

| Symptom | Cause | Resolution |
| --- | --- | --- |
| `The user 'x' is currently used by process ...` | Active session or running process | End the session, then retry, or use `--force`. |
| `/home/<user>` still present after removal | `--remove-home` not supplied | Delete manually or re-run with `--remove-home`. |
| Files still owned by old UID | `--remove-all-files` not used | `find / -uid <uid>` and reassign or delete. |
| `deluser: command not found` | Non-Debian distribution | Use [userdel](userdel.md) instead. |

## References

- `man 8 deluser`
- `man 5 deluser.conf`
- Debian Administrator's Handbook — User and Group Management

## Related
- [User-and-Group-Management](User-and-Group-Management.md) — parent topic
- [userdel](userdel.md) — low-level equivalent for removing users
- [User-Management-with-useradd-and-adduser](User-Management-with-useradd-and-adduser.md) — inverse: creating users
- [groupdel](groupdel.md) — removing groups counterpart
- [Linux Administration & Server Hardening](../Readme.md) — course hub
