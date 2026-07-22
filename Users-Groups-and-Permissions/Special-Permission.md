# Special Permission

## Overview

Beyond the standard read/write/execute triad, Linux provides three **special permission bits** that alter how files and directories behave at execution and creation time: **setuid**, **setgid**, and the **sticky bit**. They exist to solve legitimate problems — letting an unprivileged user change their own password, keeping a shared project directory group-consistent, or protecting world-writable scratch space like `/tmp` — but each also expands the attack surface. Understanding exactly what they do, how they render in `ls -l`, and how they are represented in octal is core to both administering and auditing a hardened system.

> [!IMPORTANT]
> Special bits execute code or influence file ownership with **elevated privilege**. A misconfigured setuid/setgid binary is one of the most common Linux privilege-escalation vectors. Treat every special bit as a security-relevant control, not just a convenience.

## Concepts

| Bit | Symbolic | Applies to | Effect |
| --- | --- | --- | --- |
| setuid | `u+s` | Executable files | Process runs with the **file owner's** UID (often root), not the caller's |
| setgid | `g+s` | Executable files | Process runs with the **file group's** GID |
| setgid | `g+s` | Directories | New files/subdirs **inherit the directory's group** instead of the creator's primary group |
| sticky | `+t` | Directories | Only the file owner, directory owner, or root may **delete or rename** entries |

### How they render in `ls -l`

| Position | Normal | Special set (exec present) | Special set (exec absent) |
| --- | --- | --- | --- |
| Owner (setuid) | `x` | `s` | `S` |
| Group (setgid) | `x` | `s` | `S` |
| Others (sticky) | `x` | `t` | `T` |

An uppercase `S`/`T` warns that the special bit is set but the underlying execute bit is **not**, which is usually a mistake.

## 1. Setuid (Set User ID)

- **Function:**
When set on an executable file, users run the program with the permissions of the file's owner (often root), rather than their own. This is essential for programs that must perform privileged operations.
- **How to set:**

```bash
chmod u+s <file>
```

- **Example:**
The `passwd` command has the setuid bit set, allowing users to update their passwords:

```bash
ls -l /usr/bin/passwd
```

Output example: -rwsr-xr-x 1 root root ... /usr/bin/passwd

## 2. Setgid (Set Group ID)

- **Function:**
    - **On files:** The program runs with the group permissions of the file's group owner.
    - **On directories:** Files created inside inherit the group ownership of the directory (not the user's primary group).
- **How to set:**

```bash
chmod g+s <file-or-directory>
```

- **Example:**
Setting setgid on a project directory to ensure group consistency:

```bash
chmod g+s /project/shared
```

```bash
ls -ld /project/shared
```

Output: drwxr-sr-x ...

## 3. Sticky Bit

- **Function:**
When set on a directory, only file owners, the directory owner, or root can delete or rename files within it. This is essential in shared directories like `/tmp`.
- **How to set:**

```bash
chmod +t <directory>
```

- **Example:**
The `/tmp` directory:

```bash
ls -ld /tmp
```

Output: drwxrwxrwt ...

## Checking Special Permissions

- **Using `ls -l`:**
Special permissions appear as follows:
    - **setuid**: `rws` (the `x` for user is replaced by `s`)
    - **setgid**: `rws` (the `x` for group is replaced by `s`)
    - **sticky**: `rwt` (the `x` for others is replaced by `t`)

## Numeric (Octal) Representation

- Special bits use a fourth (leading) digit:
    - `4` = setuid
    - `2` = setgid
    - `1` = sticky
- **Examples:**

- setuid + rwxr-xr-x

```bash
chmod 4755 <file>
```

- setgid + rwxr-sr-x

```bash
chmod 2775 <dir>
```

## Useful Commands

- **Find all setuid files:**

```bash
find / -perm -4000
```

## Best Practices

> [!TIP]
> - Prefer purpose-built tools (`sudo`, capabilities via `setcap`) over hand-rolled setuid binaries. `setcap cap_net_raw+ep` grants a single capability instead of full root.
> - Never place the setuid bit on shell interpreters, scripts, or anything that spawns a shell — it grants an instant root shell.
> - Keep an inventory baseline of setuid/setgid binaries and alert on drift; unexpected additions are a classic post-exploitation persistence marker.
> - Mount data partitions with `nosuid` where privileged binaries are never expected (e.g. `/home`, `/tmp`, removable media).

## Security Considerations

Be cautious: programs with setuid or setgid bits set may be security risks if not carefully coded, since they execute with elevated privileges. A bug in such a program (command injection, a `PATH`-relative call, an unsafe `system()`) can be leveraged to run arbitrary code as the file owner. Audit them regularly:

```bash
find / -perm -4000 -o -perm -2000 -type f 2>/dev/null
```

CIS Benchmarks recommend recording the authorized setuid/setgid set and reviewing any file that does not appear on it (CIS "Ensure no unowned/no world-writable files exist" and SUID/SGID audit controls).

## Related

- [Setuid(Set-User-ID)](Setuid(Set-User-ID).md) — set-user-ID bit in depth.
- [Setgid(Set-Group-ID)](Setgid(Set-Group-ID).md) — set-group-ID bit in depth.
- [Sticky-Bit](Sticky-Bit.md) — sticky bit on shared directories.
- [chmod](chmod.md) — command used to set every special bit.
- [Linux-Permissions](Linux-Permissions.md) — base permission model these bits extend.
- [Privilege-Escalation-Example-Using](Privilege-Escalation-Example-Using.md) — special bits abused for privesc.
- Privilege-Escalation — SUID/SGID misconfiguration as an attack path.
- [Linux Administration & Server Hardening](../Readme.md) — course hub.
