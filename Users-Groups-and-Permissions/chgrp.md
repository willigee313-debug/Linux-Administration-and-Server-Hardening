# chgrp

## Overview

The `chgrp` command changes the **group ownership** of files and directories in Linux. It is commonly used in shared environments where multiple users need collaborative access to the same resources through a common group. `chgrp` supports recursive operations, wildcard expansion (performed by the shell), numeric GIDs, and reference-based group assignment.

> [!NOTE]
> `chgrp` changes only the *group* of a file. To change the owner, or the owner and group together, use [chown](chown.md); to change the permission bits, use [chmod](chmod.md).

## Concepts

### Basic Syntax

```bash
chgrp [OPTIONS] GROUP FILE...
```

### Parameters

| Parameter | Description |
|---|---|
| `GROUP` | Target group name or numeric GID |
| `-R` | Recursively change the group of directories and their contents |
| `-f` | Suppress most error messages |
| `-v` | Display a message for every processed file |
| `-c` | Display changes only when a modification occurs |

## Commands

### Change Group of a File or Directory

Change the group of `log/` to `root`:

```bash
chgrp root log/
```

Change the group of `yum.log` to `root`:

```bash
chgrp root yum.log
```

### Change Group of Multiple Files

```bash
chgrp root wtmp speech-dispatcher
```

### Using Wildcards

Change the group of all files and directories in the current directory:

```bash
chgrp root *
```

Change the group of all `.log` files:

```bash
chgrp root *.log
```

Change the group of all files with an extension:

```bash
chgrp root *.*
```

Change the group of all files and directories to `armour`:

```bash
chgrp armour *
```

### Recursive Group Changes

Recursively change the group of all files and directories in the current directory:

```bash
chgrp -R root *
```

Recursively change the group to `armour`:

```bash
chgrp -R armour *
```

Recursively change the group of a specific directory:

```bash
chgrp -R infosec /var/www/html
```

### Change Group Using GID

Change the group using a numeric GID:

```bash
chgrp 1001 file.txt
```

### Copy Group Ownership from Another File

Make `file2` use the same group as `file1`:

```bash
chgrp --reference=file1 file2
```

### Verify Group Ownership

Display ownership and group information:

```bash
ls -l file.txt
```

Display numeric UID and GID:

```bash
ls -ln file.txt
```

### Verbose Output

Show every processed file:

```bash
chgrp -v root *.log
```

Show only files whose group was changed:

```bash
chgrp -c root *.log
```

## Troubleshooting

### Operation Not Permitted

```bash
chgrp root file.txt
```

```text
chgrp: changing group of 'file.txt': Operation not permitted
```

Cause:

- You do not own the file.
- You are not a member of the target group.
- You do not have sufficient privileges.

Solution — run with elevated privileges:

```bash
sudo chgrp root file.txt
```

### Invalid Group

```bash
chgrp unknowngroup file.txt
```

```text
chgrp: invalid group: 'unknowngroup'
```

Verify that the group exists:

```bash
getent group unknowngroup
```

## Best Practices

- Use `ls -l` before and after running `chgrp` to verify changes.
- Remember that wildcards (`*`, `*.log`, etc.) are expanded by the shell **before** `chgrp` executes.
- Be careful when using `-R`, especially in system directories — a wrong recursive change can break service access.
- Regular users can change a file's group only to groups they belong to.
- Root can change a file's group to any valid group.

## Security Considerations

- **Least privilege via groups**: prefer scoping shared access with a dedicated group and `chgrp` over loosening permissions with `chmod`. This keeps `others` with no access.
- **Recursive caution**: `chgrp -R` on system paths (e.g., `/var`, `/etc`) can hand unintended write paths to a group; review the target tree first.
- **Setgid interaction**: on a setgid directory, new files inherit the directory's group automatically — combine `chgrp` with the setgid bit for consistent collaborative ownership (see [Special-Permission](Special-Permission.md)).
- **Audit trail**: unexpected group ownership changes on sensitive files can indicate tampering; monitor with file-integrity tooling.

## References

| Resource | Description |
|---|---|
| `man chgrp` | Full command reference |
| `man 5 group` | `/etc/group` group definitions |
| `getent group` | Resolve group names/GIDs via NSS |

## Related

- [chown](chown.md) — change file owner (and group)
- [chmod](chmod.md) — change permission bits
- [Group-File-Linux-Group-Account-File](Group-File-Linux-Group-Account-File.md) — `/etc/group` group definitions
- [Linux-Permissions](Linux-Permissions.md) — permission model overview
- [Special-Permission](Special-Permission.md) — setgid inheritance for shared directories
- [Linux Administration & Server Hardening](../Readme.md) — course hub
