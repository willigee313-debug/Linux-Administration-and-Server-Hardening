# groupadd

The `groupadd` command creates new group accounts on a Linux system by writing a fresh record into `/etc/group` (and `/etc/gshadow`). It is the low-level, non-interactive counterpart to the Debian `addgroup` wrapper, and it is the canonical way to provision the shared groups that back file-system access control, `sudo` policy, and service isolation.

## Overview

Every Linux file and process is owned by exactly one user and one group, so groups are the primary mechanism for granting a set of accounts shared access to files, directories, and privileged operations. `groupadd` allocates the next available Group ID (GID) from the range defined in `/etc/login.defs` (`GID_MIN`/`GID_MAX`), or a GID you specify, and appends the new record. It changes only the group databases — it does not create home directories or add members. Membership is layered on afterwards with [gpasswd](gpasswd.md) or `usermod -aG`.

> [!NOTE]
> `groupadd` must be run as `root` (or through `sudo`) because it writes to files owned by `root` under `/etc`.

## Concepts

| Concept | Description |
| --- | --- |
| Primary group | The group recorded in a user's `/etc/passwd` record; owns files the user creates by default. |
| Supplementary group | Any additional group a user belongs to, listed in `/etc/group`; grants shared access without changing the primary group. |
| GID | Numeric Group ID. Regular groups draw from `GID_MIN`–`GID_MAX`; system groups from `SYS_GID_MIN`–`SYS_GID_MAX`. |
| System group | A group created with `-r`, using a low GID reserved for daemons and services rather than human users. |
| `/etc/group` | World-readable database of group name, GID, and member list. |
| `/etc/gshadow` | Root-only database of group passwords and administrators. |

## Configuration

`groupadd` reads its defaults from `/etc/login.defs`. The most relevant keys:

| Key | Purpose |
| --- | --- |
| `GID_MIN` / `GID_MAX` | GID range for regular (non-system) groups. |
| `SYS_GID_MIN` / `SYS_GID_MAX` | GID range used when `-r` is supplied. |
| `MAX_MEMBERS_PER_GROUP` | Splits oversized group lines to keep NIS-compatible entry lengths (rarely needed). |

Command-line options always override these file defaults.

## Commands

| Option | Meaning |
| --- | --- |
| `-g GID` | Assign a specific numeric GID instead of the next free one. |
| `-o` | Allow a non-unique (duplicate) GID; must accompany `-g`. |
| `-r` | Create a system group (GID from the system range). |
| `-f` | Exit successfully if the group already exists; with `-g`, pick another GID on collision. |
| `-p PASSWORD` | Set an encrypted group password (pre-hashed value, not plaintext). |
| `-K KEY=VALUE` | Override a `/etc/login.defs` default for this invocation. |
| `-h`, `--help` | Show help. |

### Display Help

- Show help information.

```bash
groupadd --help
```

- Show help using the short option.

```bash
groupadd -h
```

## Examples

### Create Groups

- Create a group named `IT`.

```bash
groupadd IT
```

- Create a group named `EMP`.

```bash
groupadd EMP
```

- Create a group named `SALES`.

```bash
groupadd SALES
```

### Create Groups with Specific GIDs

- Create a group named `demo` with GID `1030`.

```bash
groupadd -g 1030 demo
```

- Create a group named `demo2` with the same GID `1030`, allowing duplicate GIDs.

```bash
groupadd -g 1030 -o demo2
```

> [!WARNING]
> Duplicate GIDs (`-o`) make two group names resolve to the same numeric identity. The kernel enforces permissions on the GID, not the name, so both groups become interchangeable for access-control purposes. Use this only for deliberate aliasing.

### Create Groups with Passwords

> [!WARNING]
> Group passwords are rarely used on modern Linux systems and are generally discouraged. A shared secret cannot be attributed to an individual and defeats accountability. Prefer `sudo` group membership or `sg`/`newgrp` without a password.

- Generate an encrypted password hash.

```bash
openssl passwd 12345
```

- Example output.

```text
$1$b88yo2W0$rbdvkgFNK0Tr0hl/gUVD50
```

- Create group `demo3` with an encrypted password.

```bash
groupadd -p '$1$b88yo2W0$rbdvkgFNK0Tr0hl/gUVD50' demo3
```

- Create group `demo4` with an MD5-encrypted password hash.

```bash
groupadd -p '$1$lsPUtmoT$B.yzyVcDquljgIQaRk1YW.' demo4
```

> [!IMPORTANT]
> `-p` expects an already-hashed value. A plaintext string passed here is stored verbatim and will never match at authentication time. It is also visible in shell history and the process table — generate the hash separately, as shown above.

### Create a System Group

- Create a system group named `sysgrp`.

```bash
groupadd -r sysgrp
```

> [!NOTE]
> System groups take a GID from the reserved low range (`SYS_GID_MIN`–`SYS_GID_MAX`) and are intended for daemons and services rather than human users. Pairing a dedicated service account with a dedicated system group is the standard way to run a daemon with least privilege.

### Verify Group Creation

- Search for the created groups.

```bash
grep -E 'IT|EMP|SALES|demo|sysgrp' /etc/group
```

- Display the last few group entries.

```bash
tail -n 10 /etc/group
```

## Best Practices

> [!TIP]
> - Let `groupadd` auto-assign GIDs unless you must match an existing identity (e.g., an NFS export or another host); consistent GIDs across hosts prevent cross-mount permission surprises.
> - Use `-r` for every service group so daemon GIDs stay out of the human-user range.
> - Give shared-data directories a dedicated group plus the setgid bit (`chmod g+s`) so new files inherit the group automatically. See [Setgid(Set-Group-ID)](Setgid(Set-Group-ID).md).
> - Add members with `usermod -aG group user` or [gpasswd](gpasswd.md) `-a` — never edit `/etc/group` by hand while other administrators may be logged in; use `vigr` if you must edit directly.

## Security Considerations

> [!WARNING]
> - Avoid group passwords entirely (see above); they undermine per-user accountability.
> - Duplicate GIDs (`-o`) silently merge two groups' authority — audit for them with `awk -F: 'seen[$3]++{print}' /etc/group`.
> - Group membership is a privilege grant. Adding an account to `sudo`, `wheel`, `docker`, `adm`, or `disk` can be equivalent to granting root. CIS benchmarks recommend auditing membership of these high-impact groups regularly.
> - After creating a group, review who can modify it: administrators listed in `/etc/gshadow` can add members without `root`.

## Troubleshooting

| Symptom | Cause / Fix |
| --- | --- |
| `groupadd: group 'IT' already exists` | The name is taken. Choose another name, or use `-f` to exit cleanly. |
| `groupadd: GID '1030' already exists` | GID in use. Pick a free GID, let it auto-assign, or add `-o` to allow duplication (rarely wanted). |
| `groupadd: Permission denied` | Not running as `root`. Prefix with `sudo`. |
| `groupadd: cannot lock /etc/group` | Another tool holds the lock, or a stale `*.lock` file remains under `/etc`. Ensure no other user-management process is running, then remove the stale lock. |

## References

| Resource | Description |
| --- | --- |
| `man 8 groupadd` | Full manual page and option reference. |
| `man 5 login.defs` | Defaults consulted by `groupadd` (GID ranges). |
| `man 5 group` / `man 5 gshadow` | Format of the group databases. |
| CIS Linux Benchmark | Guidance on auditing high-privilege group membership. |

## Related
- [User-and-Group-Management](User-and-Group-Management.md) — parent topic
- [groupmod](groupmod.md) — modify groups after creation
- [groupdel](groupdel.md) — remove created groups
- [gpasswd](gpasswd.md) — add members and administer the group
- [Group-File-Linux-Group-Account-File](Group-File-Linux-Group-Account-File.md) — where new groups are recorded
- [Gshadow-File-Secure-Group-Access-File](Gshadow-File-Secure-Group-Access-File.md) — secure group passwords and administrators
- [Linux Administration & Server Hardening](../Readme.md) — course hub
