# /etc/shadow – Secure User Password File

## Overview

The `/etc/shadow` file stores encrypted password information and password-aging policies for local user accounts. It enhances system security by separating the sensitive password hashes from the world-readable `/etc/passwd` file.

Only the `root` user and privileged processes can read `/etc/shadow`, which keeps password hashes out of reach of ordinary users and offline cracking attempts.

> [!IMPORTANT]
> `/etc/shadow` is one of the most security-critical files on a Linux system. Its confidentiality (restrictive permissions) and integrity (no unauthorised edits) are both essential to account security.

## Concepts

The `/etc/shadow` file contains:

- Encrypted password hashes
- Password-aging information
- Password-expiration policies
- Account-expiration settings
- Account-lock status

Where `/etc/passwd` answers "who is this account?", `/etc/shadow` answers "what is its secret, and when does it expire?".

## File Permissions

View the file permissions:

```bash
ls -l /etc/shadow
```

Example output:

```text
-r--------. 1 root root 1820 Jun  9 10:00 /etc/shadow
```

The mode `-r--------` (owner read-only, no group or other access) confirms that only the `root` user can read the file.

## Format of `/etc/shadow`

Each entry consists of nine colon-separated fields:

```text
username:encrypted_password:last_change:min:max:warn:inactive:expire:reserved
```

### Field Descriptions

| Field | Name | Description |
| :-- | :-- | :-- |
| 1 | `username` | User login name |
| 2 | `encrypted_password` | Password hash or account-status indicator |
| 3 | `last_change` | Days since January 1, 1970 when the password was last changed |
| 4 | `min` | Minimum number of days before the password can be changed |
| 5 | `max` | Maximum number of days a password remains valid |
| 6 | `warn` | Days before expiration to warn the user |
| 7 | `inactive` | Days after password expiration before the account is disabled |
| 8 | `expire` | Account expiration date (days since January 1, 1970) |
| 9 | `reserved` | Reserved for future use |

## Password Field Values

The second field determines password status:

| Value | Meaning |
| :-- | :-- |
| Password hash | Valid password exists |
| `!` | Account password is locked |
| `!!` | Password has not been set |
| `*` | Password login disabled |
| Empty | No password configured |

Examples:

```text
root:$6$hashvalue...
daemon:*
sshd:!!
apache:!!
```

## Password Hash Format

Password hashes typically follow this structure:

```text
$id$salt$hashed_password
```

### Common Hash Algorithms

| Prefix | Algorithm |
| :-- | :-- |
| `$1$` | MD5 |
| `$2a$` | Blowfish |
| `$2y$` | Eksblowfish |
| `$5$` | SHA-256 |
| `$6$` | SHA-512 |

Example:

```text
$6$salt$hashed_password
```

This indicates a SHA-512 password hash.

> [!TIP]
> Prefer SHA-512 (`$6$`) or a modern algorithm such as yescrypt where the distribution supports it. MD5 (`$1$`) is cryptographically broken and must not be used for new password hashes.

## Examples

### Example Entries

```text
root:$6$TDKFHGuCcRyEEWKu$t4DBGSvhWS8Oa1Z/...:20050:0:99999:7:::
daemon:*:17110:0:99999:7:::
sshd:!!:20034::::::
apache:!!:20063::::::
armour:$6$wf/dsYGa7aWhThyi$Yl0...:20065:0:99999:7:::
```

### Example Entry Breakdown

```text
daemon:*:17110:0:99999:7:::
```

| Field | Value | Description |
| :-- | :-- | :-- |
| Username | `daemon` | System account |
| Password | `*` | Password login disabled |
| Last change | `17110` | Days since the Unix epoch |
| Min age | `0` | Password can be changed immediately |
| Max age | `99999` | Password effectively never expires |
| Warn | `7` | Warning begins 7 days before expiration |
| Inactive | Empty | Not configured |
| Expire | Empty | Not configured |

## Commands

### Viewing Password-Aging Information

Display the password-aging settings for an account:

```bash
chage -l username
```

Example:

```bash
chage -l armour
```

Sample output:

```text
Last password change                                    : Jun 09, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

### Configuring Password Aging

Set a password-aging policy:

```bash
chage -M 90 -W 7 -m 1 username
```

| Option | Description |
| :-- | :-- |
| `-M 90` | Password expires after 90 days |
| `-W 7` | Warn the user 7 days before expiration |
| `-m 1` | Minimum 1 day before the password can be changed |

Example:

```bash
chage -M 90 -W 7 -m 1 armour
```

### Locking and Unlocking Accounts

Lock an account:

```bash
passwd -l username
```

Example:

```bash
passwd -l armour
```

Unlock an account:

```bash
passwd -u username
```

Example:

```bash
passwd -u armour
```

### Checking Default Password Policies

Display password-related defaults:

```bash
grep PASS /etc/login.defs
```

Example output:

```text
PASS_MAX_DAYS   99999
PASS_MIN_DAYS   0
PASS_MIN_LEN    8
PASS_WARN_AGE   7
```

Display the hashing configuration:

```bash
grep ENCRYPT_METHOD /etc/login.defs
```

Example:

```text
ENCRYPT_METHOD SHA512
```

## Configuration

### Editing `/etc/shadow` Safely

Create a backup before editing:

```bash
cp /etc/shadow /etc/shadow.bak
```

Use the safe editor, which locks the file and validates on save:

```bash
vipw -s
```

Alternatively (only when necessary):

```bash
vim /etc/shadow
```

> [!WARNING]
> Direct editing of `/etc/shadow` should only be performed when absolutely necessary. Prefer `passwd`, `chage`, and `usermod`, which update the file atomically and keep it consistent with `/etc/passwd`.

### Validating User-Account Files

Check `/etc/passwd` and `/etc/shadow` consistency:

```bash
pwck
```

Read-only verification:

```bash
pwck -r
```

Check the group files:

```bash
grpck
```

## Related Files

| File | Purpose |
| :-- | :-- |
| `/etc/passwd` | User-account information |
| `/etc/shadow` | Password hashes and aging information |
| `/etc/group` | Group-account information |
| `/etc/gshadow` | Secure group information |
| `/etc/login.defs` | Default account and password policies |
| `/etc/security/pwquality.conf` | Password-complexity requirements |

## Security Considerations

- Never share or expose the contents of `/etc/shadow`; treat it as a credential store.
- Confirm permissions stay at `0000`/`0400`-class (root read-only) — this is a standard CIS Benchmark check. Group or world read access is a critical finding.
- Use strong password-hashing algorithms such as SHA-512 (or yescrypt where available).
- Enforce password-aging policies (`PASS_MAX_DAYS`, `PASS_WARN_AGE`) where organisational policy requires them.
- Regularly review locked (`!`) and inactive accounts, and disable those no longer needed.
- Use `passwd`, `chage`, `usermod`, and related tools instead of manually editing the file.
- Monitor for unauthorised modifications with file-integrity tooling (for example AIDE) and run `pwck` periodically.

## Summary

The `/etc/shadow` file is the secure password database used by Linux systems. It stores password hashes, password-aging information, and account-expiration settings while restricting access to privileged users. Proper management of this file — restrictive permissions, strong hashing, aging policy, and integrity monitoring — is essential for maintaining system security.

## Related

- [Passwd-File-Linux-User-Account-File](Passwd-File-Linux-User-Account-File.md) — the companion, world-readable account database.
- [passwd](passwd.md) — command that updates password hashes in this file.
- [Setuid(Set-User-ID)](Setuid(Set-User-ID).md) — how `passwd` gains write access to `/etc/shadow` via SUID.
- [Reset-the-Root-Password-in-CentOS-Stream-10](Reset-the-Root-Password-in-CentOS-Stream-10.md) — recovering when the root hash here is lost.
- [Linux Administration & Server Hardening](../Readme.md) — course hub.
