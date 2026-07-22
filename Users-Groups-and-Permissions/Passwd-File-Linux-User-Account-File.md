# /etc/passwd – Linux User Account File

## Overview

The `/etc/passwd` file is one of the core user account databases in Linux. It stores basic information about every user account and is consulted by the system to identify users and determine their login environment.

Each line in the file represents a single user account and consists of **seven colon-separated fields**. The file is world-readable by design because many programs need to translate a UID into a username (for example, `ls -l` showing file owners), but it must **never** contain password hashes on a modern system.

> [!NOTE]
> Historically, password hashes were stored in this file. Modern Linux systems store hashes in `/etc/shadow` (readable only by `root` and the `shadow` group), and the password field in `/etc/passwd` typically contains `x` as a placeholder pointing to the shadow file.

## Concepts

### What `/etc/passwd` Contains

Each account record carries:

- Username
- User ID (UID)
- Primary Group ID (GID)
- User description (GECOS field)
- Home directory
- Default login shell

### Record Structure

```mermaid
flowchart LR
    A["username"] --> B["x"] --> C["UID"] --> D["GID"] --> E["comment (GECOS)"] --> F["home directory"] --> G["login shell"]
```

## Architecture

### File Format

Each entry follows this format:

```text
username:x:UID:GID:comment:home_directory:shell
```

### Field Descriptions

| Field Number | Field Name | Description |
|---|---|---|
| 1 | `username` | User login name |
| 2 | `x` | Indicates that the password hash is stored in `/etc/shadow` |
| 3 | `UID` | User ID |
| 4 | `GID` | Primary Group ID |
| 5 | `comment` | GECOS field containing user information |
| 6 | `home_directory` | User's home directory |
| 7 | `shell` | User's default login shell |

### Example Entry

```text
root:x:0:0:root:/root:/bin/bash
```

#### Breakdown

| Field | Value | Description |
|---|---|---|
| Username | `root` | Superuser account |
| Password | `x` | Password stored in `/etc/shadow` |
| UID | `0` | Root user ID |
| GID | `0` | Root group ID |
| Comment | `root` | User description |
| Home Directory | `/root` | Root user's home directory |
| Shell | `/bin/bash` | Default login shell |

## Concepts — Common User Types

### Root User

```text
root:x:0:0:root:/root:/bin/bash
```

- UID: `0`
- Full administrative privileges

### Service Accounts

```text
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
```

- Used by system services
- Typically assigned non-login shells

### Regular Users

```text
armour:x:1000:1000:Armour:/home/armour:/bin/bash
```

- Usually assigned UIDs starting from `1000`
- Have home directories and interactive shells

> [!NOTE]
> The exact UID boundary between system and regular accounts is defined by `UID_MIN`/`UID_MAX` in `/etc/login.defs` (commonly `1000`–`60000`). Some distributions reserve the `100`–`999` range for system/service accounts.

## Commands

### Viewing `/etc/passwd`

Display all entries:

```bash
cat /etc/passwd
```

Display with pagination:

```bash
less /etc/passwd
```

Display a specific user:

```bash
grep '^armour:' /etc/passwd
```

Using `getent` (queries all configured name sources, not just the flat file):

```bash
getent passwd armour
```

### Listing Regular Users

Display users with UID 1000 or higher:

```bash
awk -F: '$3>=1000 && $3<65534 {print $1}' /etc/passwd
```

Display usernames and shells:

```bash
awk -F: '{print $1, $7}' /etc/passwd
```

### Checking User Information

Display current user information:

```bash
id
```

Display account details from `/etc/passwd`:

```bash
getent passwd $(whoami)
```

### Editing `/etc/passwd`

Create a backup before making changes:

```bash
cp -v /etc/passwd /etc/passwd.bak
```

Use the recommended editor (locks the file and validates on save):

```bash
vipw
```

Alternatively (not recommended — no locking or validation):

```bash
vim /etc/passwd
```

### Validating the File

Check for syntax errors:

```bash
pwck
```

Read-only validation:

```bash
pwck -r
```

## Examples

### Example `/etc/passwd` Entries

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:2:2:daemon:/sbin:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
armour:x:1000:1000:Armour:/home/armour:/bin/bash
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: Terminal showing cat /etc/passwd output with the root, service, and regular user entries and their seven colon-separated fields_

## Best Practices

- Use `useradd`, `usermod`, and `userdel` for account management rather than hand-editing.
- Use `vipw` instead of directly editing `/etc/passwd` — it locks the file and prevents concurrent-edit corruption.
- Maintain a backup before modifications.
- Ensure every user has a **unique UID**.
- Reserve UID `0` for the root account only.
- Assign `/sbin/nologin` or `/usr/sbin/nologin` to service accounts that do not require interactive access.
- Periodically review and remove unused accounts.

## Security Considerations

- **A writable `/etc/passwd` is a critical privilege-escalation vector.** If an attacker can write to it, they can add a UID 0 account or replace the `x` placeholder with a crafted password hash to gain root. The file should be owned by `root:root` with mode `644`.
- **No duplicate UID 0.** Any account other than `root` with UID `0` is effectively a hidden root account — audit for it with `awk -F: '($3==0){print $1}' /etc/passwd`.
- **Password field must stay `x`.** A blank field 2 means the account has no password; audit for it.
- `/etc/passwd` must remain readable by all users, but its contents (GECOS, home paths, shells) can aid an attacker's enumeration — do not store sensitive data in the comment field.
- Incorrect modifications can prevent user logins and affect system functionality; always validate with `pwck` after manual edits.

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| User cannot log in after edit | Malformed field or wrong shell path | Run `pwck`; restore from `/etc/passwd.bak` |
| `getent passwd user` returns nothing | Account only in a remote source (LDAP) not reachable | Check `nsswitch.conf` and the directory service |
| "cannot lock /etc/passwd" | Another edit in progress or stale lock | Ensure no `vipw`/`useradd` is running; remove stale `/etc/passwd.lock` |
| Unexpected root account | Duplicate UID 0 injected | Remove the rogue entry; investigate for compromise |

## References

| Resource | Description |
|---|---|
| `man 5 passwd` | Format of the password file |
| `man vipw` | Safely edit `/etc/passwd` and `/etc/shadow` |
| `man pwck` | Verify integrity of password files |
| `/etc/login.defs` | UID range and account-creation defaults |

## Related

- [Linux Administration & Server Hardening](../Readme.md) — course hub.
- [User-and-Group-Management](User-and-Group-Management.md) — parent topic covering account lifecycle.
- [Shadow-File-Secure-User-Passwords-File](Shadow-File-Secure-User-Passwords-File.md) — paired file holding password hashes.
- [Group-File-Linux-Group-Account-File](Group-File-Linux-Group-Account-File.md) — companion group account file.
- [User-Management-with-useradd-and-adduser](User-Management-with-useradd-and-adduser.md) — creating accounts the supported way.
- [Real-UID-vs-Effective-UID-in-Linux](Real-UID-vs-Effective-UID-in-Linux.md) — how UIDs drive process identity and permission checks.
