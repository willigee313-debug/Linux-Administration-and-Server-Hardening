# chmod

## Overview

The `chmod` command modifies the **permissions** of files and directories in Linux. Permissions govern who can read, write, and execute a file, and are the cornerstone of the Unix discretionary access-control model. `chmod` accepts two equivalent notations: a **symbolic** method (clear for ad-hoc edits) and a **numeric/octal** method (concise and common in scripts).

> [!IMPORTANT]
> Over-permissive files are a recurring security finding. World-writable files (`o+w`), and especially `777` permissions, allow any local user to modify data or code — a frequent stepping stone in privilege escalation.

## Concepts

### Symbolic Method

Permissions:

- `r` = **read**
- `w` = **write**
- `x` = **execute**

Scope:

- `u` = **user/owner**
- `g` = **group**
- `o` = **others**
- `a` = **all** (`u+g+o`)

Operators: `+` adds, `-` removes, `=` sets exactly (clearing any bit not listed).

### Numeric (Octal) Method

- **4** = read (r)
- **2** = write (w)
- **1** = execute (x)

Add the values for each scope, in the order **user (u)**, **group (g)**, **others (o)**.

| Number | Permissions | Meaning |
| :-- | :-- | :-- |
| 7 | rwx | all |
| 6 | rw- | read, write |
| 5 | r-x | read, exec |
| 4 | r-- | read only |
| 0 | --- | none |

## Commands

### Common Symbolic Usage

Add **write** for others:

```bash
chmod o+w password-armour
```

Remove **write** from others:

```bash
chmod o-w password-armour
```

Add **read, write** for group:

```bash
chmod g+rw password-armour
```

Remove **write** from group:

```bash
chmod g-w password-armour
```

Add **execute** for owner:

```bash
chmod u+x password-armour
```

Set owner permission to **read, write, execute**:

```bash
chmod u+rwx password-armour
```

Remove **all permissions** from owner:

```bash
chmod u-rwx password-armour
```

Remove **all permissions** from everyone:

```bash
chmod ugo-rwx password-armour
```

Add **all permissions** for everyone:

```bash
chmod ugo+rwx password-armour
```

Equivalent — add **all permissions** to all users:

```bash
chmod a+rwx password-armour
```

Remove **all permissions** for all users:

```bash
chmod a-rwx password-armour
```

Owner: rwx, Group: rw, Others: r:

```bash
chmod u+rwx,g+rw,o+r password-armour
```

Set group permission to **read, write** (removes execute if present):

```bash
chmod g=rw password-armour
```

All users: **read, write** only:

```bash
chmod a=rw password-armour
```

Owner & group: rwx, Others: r:

```bash
chmod u=rwx,g=rwx,o=r password-armour
```

Add **execute** for all users (shorthand for `a+x`):

```bash
chmod +x password-armour
```

Directory `log/` — all users get rwx:

```bash
chmod a=rwx log/
```

Owner: rwx, Group/Others: rx:

```bash
chmod u=rwx,go=rx log/
```

**Recursively** set all permissions for everyone under `log/`:

```bash
chmod -R a=rwx log/
```

### Common Numeric Usage

Only owner: **rwx**; no one else has permissions:

```bash
chmod 700 password-armour
```

Owner: rwx, Group: r, Others: r:

```bash
chmod 744 password-armour
```

Owner: rwx, Group: rx, Others: rx (*typical for scripts, programs*):

```bash
chmod 755 password-armour
```

**All** have rwx (not recommended for security):

```bash
chmod 777 password-armour
```

**No one** has any access:

```bash
chmod 000 password-armour
```

Directory: Owner: rwx, Group/Others: rx:

```bash
chmod 755 log/
```

Verbosely and recursively set **rwx** for all under `log/`:

```bash
chmod -vR 777 log/
```

Verbosely and recursively set **rwxr-xr-x** for all under `log/`:

```bash
chmod -vR 755 log/
```

Verbosely set **owner: rwx, others: none** for all `.log` files:

```bash
chmod -v 700 *.log
```

Set **744** permissions on the listed files:

```bash
chmod 744 messages boot.log cron firewalld
```

## Examples

### Symbolic vs Numeric Equivalents

| Numeric | Symbolic (`=`) | Owner | Group | Others |
|---|---|---|---|---|
| `600` | `u=rw,go=` | rw- | --- | --- |
| `644` | `u=rw,go=r` | rw- | r-- | r-- |
| `700` | `u=rwx,go=` | rwx | --- | --- |
| `755` | `u=rwx,go=rx` | rwx | r-x | r-x |
| `775` | `u=rwx,g=rwx,o=rx` | rwx | rwx | r-x |
| `777` | `a=rwx` | rwx | rwx | rwx |

> [!TIP]
> On directories, the execute bit (`x`) means "may traverse into it". A directory with read but no execute lets you list names but not access the entries — a common source of confusing "Permission denied" errors.

## Best Practices

- Use `ls -l` to verify permissions before and after a change.
- Symbolic notation is clearer for ad-hoc changes; numeric is concise and common in scripts.
- Double-check the target before `chmod -R ...` on directories — recursive changes are hard to undo.
- Avoid granting write to `others`; prefer group-based sharing (`chgrp` + group write).
- Reserve `755`/`644` as sensible defaults for executables and data files respectively.

## Security Considerations

- **World-writable is dangerous**: `o+w` on files or directories lets any local user tamper with content; on a directory it also allows deleting others' files unless the sticky bit is set.
- **777 is almost never correct**: it is a top hardening finding and a classic local privilege-escalation enabler when applied to scripts, cron jobs, or web roots.
- **Executables and SUID**: pairing broad write with the SUID bit is critical — a writable SUID-root binary is an instant root. See [Special-Permission](Special-Permission.md).
- **Least privilege**: grant the minimum bits required; audit sensitive paths with `find / -perm -0002` (world-writable) and `find / -perm -4000` (SUID).
- **Recursive scope**: `chmod -R 777` on a directory tree frequently strips needed restrictions from configuration and key files.

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| "Permission denied" entering a directory | Missing `x` (execute/traverse) bit | Add with `chmod +x dir` or `u+x` |
| Script won't run | Missing execute bit | `chmod +x script.sh` |
| Change appears ignored on a mount | Filesystem mounted `noexec`/read-only | Check `mount` options; remount if appropriate |
| Recursive change broke a service | `chmod -R` altered config/key perms | Restore from backup or reset known-good modes |

## References

| Resource | Description |
|---|---|
| `man chmod` | Full command reference |
| `stat` | Display detailed permission information |
| `umask` | Default permissions for newly created files |

## Related

- [chown](chown.md) — change file ownership
- [chgrp](chgrp.md) — change group ownership
- [Linux-Permissions](Linux-Permissions.md) — permission model overview
- [Special-Permission](Special-Permission.md) — set SUID/SGID/sticky bits
- Privilege-Escalation — weak permissions enable privilege escalation
- [Linux Administration & Server Hardening](../Readme.md) — course hub
