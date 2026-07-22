# chown

## Overview

The `chown` command changes the **owner and/or group** of files and directories in Linux. Ownership determines which user the permission bits for `u` (owner) apply to, and is therefore central to the discretionary access-control model. `chown` can modify the owner, the group, or both at once, supports recursive operations, and can copy ownership from a reference file.

> [!NOTE]
> `chown` sets *who owns* a file; [chmod](chmod.md) sets *what the owner (and others) may do*. The two are used together to design correct, least-privilege access.

## Concepts

### Basic Syntax

```bash
chown [OPTIONS] OWNER[:GROUP] FILE...
```

### Parameters

| Parameter | Description |
|---|---|
| `OWNER` | Username or UID to assign as the file owner |
| `GROUP` | Group name or GID to assign as the file group |
| `-R` | Apply changes recursively to directories and their contents |
| `-f` | Suppress most error messages |

> [!TIP]
> The `OWNER:GROUP` syntax is flexible: `chown user file` sets owner only, `chown user:group file` sets both, `chown :group file` sets group only, and `chown user: file` sets the group to the user's login group.

## Commands

### Change File Owner

Change the owner of a file to `armour`:

```bash
chown armour test.txt
```

Change the owner of `log/messages` to `root`:

```bash
chown root log/messages
```

### Recursive Ownership Changes

Recursively change ownership of `log/` and all its contents to `armour`:

```bash
chown -R armour log/
```

The position of `-R` does not matter:

```bash
chown armour -R log/
```

Recursively change ownership and suppress error messages:

```bash
chown armour -f -R log/
```

### Change Multiple Files

Change ownership of multiple files:

```bash
chown armour boot.log cron firewalld
```

### Using Wildcards

Change ownership of all `.log` files in the current directory:

```bash
chown armour *.log
```

Change ownership of all files matching `maillog*`:

```bash
chown root maillog*
```

Change ownership of all files ending with `.old`:

```bash
chown armour *.old
```

Change ownership of all files starting with `a`:

```bash
chown armour a*
```

### Change Owner and Group Together

Set owner and group to `armour`:

```bash
chown armour:armour log/
```

Recursively apply owner and group changes:

```bash
chown -R armour:armour log/
```

Equivalent command (option order does not matter):

```bash
chown armour:armour -R log/
```

### Change Group Only

Change only the group while keeping the current owner unchanged:

```bash
chown :armour log/
```

Recursively change only the group:

```bash
chown -R :armour log/
```

Recursively change the group to `infosec`:

```bash
chown -R :infosec log/
```

Equivalent command:

```bash
chown :armour -R log/
```

### Change Ownership Using UID and GID

Change owner using a UID:

```bash
chown 1001 file.txt
```

Change owner and group using UID and GID:

```bash
chown 1001:1001 file.txt
```

```bash
chown -R 1010:1011 log
```

### Copy Ownership from Another File

Set ownership of `file2` to match `file1`:

```bash
chown --reference=file1 file2
```

### Verify Ownership

Display file ownership information:

```bash
ls -l file.txt
```

Display numeric UID and GID:

```bash
ls -ln file.txt
```

## Best Practices

- Use `ls -l` before and after running `chown` to verify changes.
- Remember that wildcards (`*`) are expanded by the shell **before** `chown` runs.
- Be careful when using `-R`, especially on system directories.
- Only the root user can change ownership to another user.
- Regular users can usually change the group only to groups they belong to.

## Security Considerations

- **Ownership is authority**: giving a user ownership of a file lets them re-permission it at will (`chmod`). Assigning ownership of sensitive system files to a non-root user is effectively a privilege delegation.
- **Recursive hazards**: `chown -R` following symlinks or applied to the wrong tree (e.g., `/` or `/etc`) can hand system files to an unprivileged user and break the OS. Prefer `chown -R --no-dereference` where symlinks may be present.
- **SUID/SGID reset**: changing a file's owner clears its setuid/setgid bits by design — re-apply deliberately and only where justified (see [Special-Permission](Special-Permission.md)).
- **Attack indicator**: unexpected ownership changes on binaries, cron files, or web roots can signal compromise; monitor with file-integrity tooling.
- **Least privilege**: scope shared access through group ownership (`chown :group` or [chgrp](chgrp.md)) rather than reassigning individual user ownership.

## Troubleshooting

### Operation Not Permitted

```bash
chown armour file.txt
```

```text
chown: changing ownership of 'file.txt': Operation not permitted
```

Cause — the current user lacks permission to change ownership (changing owner requires root).

Solution — run with elevated privileges:

```bash
sudo chown armour file.txt
```

### Invalid User

```bash
chown unknownuser file.txt
```

```text
chown: invalid user: 'unknownuser'
```

Cause — the specified user does not exist.

Verify the account:

```bash
getent passwd unknownuser
```

## References

| Resource | Description |
|---|---|
| `man chown` | Full command reference |
| `man 5 passwd` | User account database format |
| `getent passwd` | Resolve users/UIDs via NSS |

## Related

- [chmod](chmod.md) — change permission bits
- [chgrp](chgrp.md) — change group ownership
- [Linux-Permissions](Linux-Permissions.md) — permission model overview
- [Access-Control-List(ACL)](Access-Control-List(ACL).md) — fine-grained ownership rules
- [Special-Permission](Special-Permission.md) — SUID/SGID/sticky bits reset by ownership change
- Privilege-Escalation — ownership misconfiguration enables privilege escalation
- [Linux Administration & Server Hardening](../Readme.md) — course hub
