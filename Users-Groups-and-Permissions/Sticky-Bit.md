# Sticky Bit

## Overview

The **sticky bit** is a special permission set almost exclusively on **directories**. When present, it restricts deletion and renaming inside that directory so that only the **file's owner**, the **directory's owner**, or **root** may remove or rename an entry — even if the directory itself is world-writable. This is what makes a shared scratch space like `/tmp` safe: everyone can create files there, but no one can delete another user's files.

> [!NOTE]
> On modern Linux the sticky bit has **no effect on regular files** — historically it hinted the kernel to keep an executable's text segment in swap, but that behaviour is obsolete. Its only meaningful use today is protecting world-writable directories.

## Concepts

- Set on a directory, the sticky bit means: *write access to the directory alone does not grant the right to delete or rename entries you do not own.*
- It occupies the **others' execute** position in the permission string:
    - `t` — sticky bit set **and** others have execute (the normal case).
    - `T` — sticky bit set but others lack execute (usually a misconfiguration).
- In octal, the sticky bit is the value **`1`** in the leading (fourth) digit.

| Symptom in `ls -l` | Meaning |
| --- | --- |
| `drwxrwxrwt` | World-writable directory, sticky bit active (correct for `/tmp`) |
| `drwxrwxrwx` | World-writable **without** sticky — any user can delete any file (dangerous) |
| `drwxrwx--T` | Sticky set but others cannot traverse/execute |

## Commands

- Grant full permissions (read, write, execute for all) on `data/`:

```bash
chmod 777 data/
```

- Check permissions for `data` directory:

```bash
ls -lh | grep data
```

- Add the sticky bit to `data/` for others:

```bash
chmod o+t data/
```

- Verify sticky bit is set (`t` appears in others' execute place):

```bash
ls -lh | grep data
```

- Remove sticky bit from `data/`:

```bash
chmod o-t data/
```

- Verify sticky bit removal:

```bash
ls -lh | grep data
```

- Set permissions to include sticky bit using octal notation `1` leading digit (sticky + 777):

```bash
chmod 1777 data/
```

- Check permissions—should show sticky bit `t`:

```bash
ls -lh | grep data
```

- Reset permissions to 777 without sticky bit:

```bash
chmod 0777 data/
```

- Check permissions to confirm sticky bit is removed:

```bash
ls -lh | grep data
```

- Set all permission bits including sticky bit and SUID/GUID bits (leading `7` = 4+2+1):

```bash
chmod 7777 data/
```

- Verify all special bits are set (`rwsrwsrwt` style permissions):

```bash
ls -lh | grep data
```

## Examples

The canonical real-world example is `/tmp`:

```bash
ls -ld /tmp
```

The expected output `drwxrwxrwt` confirms the directory is world-writable **and** sticky-protected, so users cannot tamper with each other's temporary files.

## Notes

- The leading number in `chmod` specifies special bits:
    - `4` = setuid
    - `2` = setgid
    - `1` = sticky bit
- `chmod 1777` means read/write/execute for all + sticky bit (common on `/tmp`)
- `chmod 7777` sets all three special bits (setuid, setgid, sticky) plus full permissions—usually unnecessary and potentially insecure.

## Best Practices

> [!TIP]
> - Set the sticky bit on **every** world-writable directory. Audit for offenders with:
>
> ```bash
> find / -type d -perm -0002 ! -perm -1000 2>/dev/null
> ```
>
> - Prefer per-user directories over shared world-writable ones where possible; the sticky bit mitigates but does not eliminate the risks of shared write access.
> - Avoid `chmod 7777` — combining setuid/setgid with world-writable permissions is almost always a security defect.

## Security Considerations

A world-writable directory **without** the sticky bit lets any user delete or rename other users' files, enabling denial-of-service and file-swapping attacks (for example, replacing a file another process is about to read). CIS Benchmarks explicitly require the sticky bit on all world-writable directories. Even with the sticky bit, a world-writable directory is still a place attackers can drop payloads, so keep such directories to the minimum required.

## Related

- [Setuid(Set-User-ID)](Setuid(Set-User-ID).md) — set-user-ID bit (`4`).
- [Setgid(Set-Group-ID)](Setgid(Set-Group-ID).md) — set-group-ID bit (`2`).
- [Special-Permission](Special-Permission.md) — overview of all three special permission bits.
- [chmod](chmod.md) — command used to set the sticky bit.
- [Linux-Permissions](Linux-Permissions.md) — base permission model.
- [Linux Administration & Server Hardening](../Readme.md) — course hub.
