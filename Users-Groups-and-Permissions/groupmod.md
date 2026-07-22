# groupmod

The `groupmod` command modifies existing group accounts on a Linux system — changing a group's GID, renaming it, or setting its password — by rewriting the relevant record in `/etc/group` and `/etc/gshadow`. It sits between [groupadd](groupadd.md) (create) and [groupdel](groupdel.md) (remove) in the group life cycle.

## Overview

`groupmod` edits the attributes of a group that already exists. The two operations you will reach for most are changing the numeric GID (`-g`) and renaming the group (`-n`). Both are metadata changes to the group record; neither rewrites the ownership of files on disk. That distinction matters most for a GID change, because existing files keep their **old** numeric GID and must be re-owned separately. The command runs as `root` (or via `sudo`).

> [!IMPORTANT]
> Renaming a group with `-n` keeps the same GID, so file ownership is preserved automatically. Changing the GID with `-g` does **not** update files already on disk — they retain the old GID until you `chgrp` them.

## Concepts

| Concept | Description |
| --- | --- |
| GID change (`-g`) | Assigns a new numeric identity; files owned by the old GID are left behind and must be re-owned. |
| Rename (`-n`) | Changes the name only; the GID and all file ownership stay intact. |
| Non-unique GID (`-o`) | Permits two groups to share a GID; must accompany `-g`. |
| Group password (`-p`) | Sets an encrypted group password in `/etc/gshadow` (discouraged). |

## Commands

| Option | Meaning |
| --- | --- |
| `-g GID` | Assign a new numeric GID to the group. |
| `-o` | Allow a non-unique (duplicate) GID; must be used with `-g`. |
| `-n NEW_NAME` | Rename the group, preserving its GID. |
| `-p PASSWORD` | Set an encrypted (pre-hashed) group password. |
| `-h`, `--help` | Show help. |

### Display Help

- Show help information for the `groupmod` command.

```bash
groupmod --help
```

## Examples

### Change Group ID (GID)

- Change the GID of group `demo` to `1028`.

```bash
groupmod -g 1028 demo
```

- Change the GID of group `demo` to `1030`, allowing a non-unique GID.

```bash
groupmod -o -g 1030 demo
```

> [!WARNING]
> The `-o` option permits duplicate GIDs. Use it only when intentionally sharing a GID between multiple groups.

> [!NOTE]
> After a GID change, files created under the old GID are not updated automatically. Re-own them:
> ```bash
> find / -gid 1028 -exec chgrp demo {} + 2>/dev/null
> ```

### Rename a Group

- Rename group `demo` to `DEMO1`.

```bash
groupmod -n DEMO1 demo
```

### Change Group Password

> [!WARNING]
> Group passwords are rarely used on modern Linux systems and are generally not recommended. A shared secret cannot be tied to an individual, which defeats accountability.

- Set the encrypted password of group `demo2`.

```bash
groupmod -p '$1$lsPUtmoT$B.yzyVcDquljgIQaRk1YW.' demo2
```

> [!IMPORTANT]
> `-p` expects an already-hashed value, not plaintext. A cleartext string is stored verbatim and never authenticates. It is also exposed in shell history and the process list.

### Verify Changes

- Display group information.

```bash
grep 'DEMO1' /etc/group
```

- Verify group entries in `/etc/gshadow`.

```bash
grep 'DEMO1' /etc/gshadow
```

- Display specific groups.

```bash
grep -E 'demo|demo2|DEMO1' /etc/group /etc/gshadow
```

## Best Practices

> [!TIP]
> - Prefer renaming (`-n`) over deleting and recreating a group — the GID and all file ownership survive intact.
> - When you must change a GID, plan a `chgrp` sweep of affected paths in the same maintenance window to avoid orphaned files.
> - Do not run `groupmod` on a group that owns files belonging to a running service without stopping the service first; in-flight processes cache credentials at start-up.
> - Manage membership with [gpasswd](gpasswd.md) or `usermod -aG`, not `groupmod` — `groupmod` changes group attributes, not the member list.

## Security Considerations

> [!WARNING]
> - A GID change can leave orphaned files owned by a now-unassigned number; if that number is later reused, the new group inherits access to those files. Re-own leftover files immediately.
> - Avoid group passwords (`-p`); they break per-user accountability. Grant access through membership and `sudo` instead.
> - Renaming a privileged group (`sudo`, `wheel`, `docker`) can break PAM, `sudoers`, and unit files that reference it by name — grep configuration for the old name before renaming.
> - Duplicate GIDs (`-o`) merge two groups' authority under one number; audit with `awk -F: 'seen[$3]++{print}' /etc/group`.

## Troubleshooting

| Symptom | Cause / Fix |
| --- | --- |
| `groupmod: group 'demo' does not exist` | Misspelled or already renamed. Confirm the current name in `/etc/group`. |
| `groupmod: GID '1030' already exists` | Target GID is taken. Choose a free GID or add `-o` to allow duplication (rarely wanted). |
| `groupmod: group 'DEMO1' already exists` | The new name is taken. Pick a different name. |
| `groupmod: cannot lock /etc/group` | Another user-management tool holds the lock, or a stale `*.lock` remains under `/etc`. Ensure nothing else is running, then clear the stale lock. |
| Files show a numeric group after a GID change | Expected — old files keep the old GID. Re-own with `chgrp`/`find … -exec chgrp`. |

## References

| Resource | Description |
| --- | --- |
| `man 8 groupmod` | Full manual page and option reference. |
| `man 5 group` / `man 5 gshadow` | Format of the group databases. |
| `man 1 chgrp` | Re-owning files after a GID change. |

## Related
- [User-and-Group-Management](User-and-Group-Management.md) — parent topic
- [groupadd](groupadd.md) — create groups to modify
- [groupdel](groupdel.md) — remove groups
- [gpasswd](gpasswd.md) — manage group membership and administrators
- [Group-File-Linux-Group-Account-File](Group-File-Linux-Group-Account-File.md) — the group record groupmod edits
- [Gshadow-File-Secure-Group-Access-File](Gshadow-File-Secure-Group-Access-File.md) — secure group passwords and administrators
- [Linux Administration & Server Hardening](../Readme.md) — course hub
