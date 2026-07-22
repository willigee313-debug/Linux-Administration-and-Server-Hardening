# passwd

## Overview

The `passwd` command manages user passwords and password-aging information on Linux systems. A regular user can change their own password; the superuser (`root`) can change any account's password, lock or unlock accounts, delete passwords, and force password changes at next login.

Password data is split across two files: the world-readable `/etc/passwd` (account metadata, with an `x` placeholder in the password field) and the root-only `/etc/shadow` (the actual salted password hash plus aging fields). The `passwd` command is a set-user-ID (SUID) `root` binary, which is how an unprivileged user is allowed to update their own hash in the protected shadow file.

> [!NOTE]
> On a modern system, `passwd` writes hashes using the algorithm configured in PAM (`/etc/pam.d/`) and `/etc/login.defs` — typically `yescrypt` or SHA-512. The `openssl passwd` examples below are for understanding hash formats and interoperability, not the day-to-day way to set a real user's password.

## Concepts

### Where password data lives

| File | Readable by | Holds |
| :-- | :-- | :-- |
| `/etc/passwd` | all users | username, UID, GID, GECOS, home dir, login shell (password field is `x`) |
| `/etc/shadow` | root only | salted password hash and aging fields |
| `/etc/group` | all users | group definitions and membership |
| `/etc/gshadow` | root only | secure group data and group passwords |

### `/etc/shadow` aging fields

The nine colon-separated fields control the password lifecycle:

| Field | Meaning |
| :-- | :-- |
| `username` | account name (must match `/etc/passwd`) |
| `encrypted_password` | salted hash (`!`/`*` = locked/no login) |
| `last_change` | days since epoch of last password change |
| `min` | minimum days before the password may be changed again |
| `max` | maximum days the password is valid |
| `warn` | days of warning before expiry |
| `inactive` | days after expiry before the account is disabled |
| `expire` | days since epoch when the account expires |
| `reserved` | reserved for future use |

### Hash algorithm identifiers

The `$id$` prefix of a shadow hash names the algorithm:

| Prefix | Algorithm | Notes |
| :-- | :-- | :-- |
| `$1$` | MD5 | legacy, weak — avoid |
| `$5$` | SHA-256 | acceptable |
| `$6$` | SHA-512 | strong, common default |
| `$y$` | yescrypt | modern default on current distros |
| (none) | DES `crypt` | 8-char limit, obsolete |

## Commands

- Search for entries related to `root` in account and group databases.

```bash
grep root /etc/passwd /etc/shadow /etc/group /etc/gshadow
```

- Display the last 4 lines of user and group files with file names.

```bash
tail -v -n 4 /etc/passwd /etc/shadow /etc/group /etc/gshadow
```

- Display command help.

```bash
passwd --help
```

### Change passwords

- Change the password for the currently logged-in user.

```bash
passwd
```

- Change the password for user `u1`.

```bash
passwd u1
```

### Delete, lock, and unlock passwords

- Delete the password of user `u1`. This may allow passwordless login if permitted by the authentication configuration.

```bash
passwd -d u1
```

- Lock user `u1`'s password, preventing password-based login.

```bash
passwd -l u1
```

- Unlock user `u1`'s password.

```bash
passwd -u u1
```

- Expire the password and unlock the account, forcing a password change at the next login.

```bash
passwd -f -u u1
```

> [!WARNING]
> `passwd -d` removes the password entirely. Combined with a PAM stack that permits empty passwords, this can create a passwordless (unauthenticated) login. Lock accounts with `passwd -l` (or `usermod -L`) instead of deleting the password.

## View password status

- Show password status information for user `u9`.

```bash
passwd -S u9
```

> Example output:

```text
u9 PS 2026-06-15 0 99999 7 -1
```

The status line reads: *username*, *status code*, *last change date*, *min*, *max*, *warn*, *inactive*.

- Common status values:

| Status | Meaning |
|----------|---------|
| P | Password is set |
| NP | No password is set |
| L | Password is locked |

## Generate Password Hashes

> [!NOTE]
> These `openssl passwd` recipes illustrate hash formats. To set a real account password, prefer interactive `passwd` so the system uses its configured (strong) algorithm.

### MD5 Hash

- Generate an MD5 password hash interactively.

```bash
openssl passwd -1
```

- Generate an MD5 password hash with a custom salt.

```bash
openssl passwd -1 -salt asdfgh
```

### DES Hash

- Generate a traditional DES hash using a custom salt.

```bash
openssl passwd -crypt -salt asdfgh 123456
```

- Generate a DES hash for the password `12345`.

```bash
openssl passwd 12345
```

> [!WARNING]
> DES `crypt` truncates passwords to 8 characters and is trivially cracked; MD5 (`$1$`) is also considered broken for password storage. Use SHA-512 (`-6`) or yescrypt where a hash must be generated manually.

## Set an Encrypted Password

- Assign a pre-generated encrypted password to user `user3`.

```bash
usermod -p '$1$JafiDMw0$Q6VF2IqPINwOG6CvmKV.V0' user3
```

> [!TIP]
> Always single-quote the hash. Characters such as `$` would otherwise be expanded by the shell, corrupting the stored value.

## Inspect Account Files

### `/etc/passwd`

- View or edit user account records.

```bash
vim /etc/passwd
```

> Example entry:

```text
armour:x:1000:1000:Armour:/home/armour:/bin/bash
```

> Field format:

```text
username:password:UID:GID:GECOS:home_directory:login_shell
```

### `/etc/shadow`

- View or edit encrypted password records (root only).

```bash
vim /etc/shadow
```

> Example entry:

```text
armour:$1$asdfgh$hEED8pSklSwWdxYbj0nFQ1:20254:0:99999:7:::
```

> Field format:

```text
username:encrypted_password:last_change:min:max:warn:inactive:expire:reserved
```

> [!IMPORTANT]
> Never edit `/etc/passwd`, `/etc/shadow`, or `/etc/group` directly with a plain editor on a production system — a syntax error can lock everyone out. Use `vipw`/`vigr` (which lock the files and validate on save) or the dedicated tools (`passwd`, `usermod`, `chage`).

## Best Practices

- Enforce password aging (`chage`, or the `min`/`max`/`warn` fields) rather than static, never-expiring passwords.
- Set strong composition and length policy through `pam_pwquality`/`pam_cracklib` in the PAM stack.
- Lock unused or service accounts (`passwd -l`) and give them a non-login shell (`/usr/sbin/nologin`).
- Prefer the system default hashing (yescrypt/SHA-512); never introduce DES or MD5 hashes.

## Security Considerations

- `/etc/shadow` must remain mode `0640` (or `0600`) owned by `root:shadow`. World-readable shadow files are a direct offline-cracking exposure.
- A password-less account (`passwd -d`) or an account with `::` in the shadow hash field can be an authentication-bypass finding.
- The `passwd` binary is SUID root; audit any custom SUID copies and monitor for tampering.
- During an engagement, dumped `/etc/shadow` hashes feed offline cracking (John the Ripper, Hashcat) — the hash prefix reveals the algorithm and therefore the cracking difficulty.

## Troubleshooting

| Symptom | Likely cause / fix |
| :-- | :-- |
| "Authentication token manipulation error" | password fails PAM quality rules, or `/etc/shadow` permissions/ownership are wrong |
| User cannot change password | `min` aging field not yet elapsed (check `chage -l <user>`) |
| Set hash does not authenticate | hash was not single-quoted, or the algorithm is not supported by the PAM config |
| Locked account still logs in | key-based SSH or another PAM factor bypasses the locked password; also disable the key/shell |

## Related

- [Shadow-File-Secure-User-Passwords-File](Shadow-File-Secure-User-Passwords-File.md) — stores the hashes `passwd` sets
- [Passwd-File-Linux-User-Account-File](Passwd-File-Linux-User-Account-File.md) — the account records `passwd` manages
- [usermod](usermod.md) — also adjusts account credentials and aging
- [gpasswd](gpasswd.md) — group-password counterpart
- [User-and-Group-Management](User-and-Group-Management.md) — parent topic
- Privilege-Escalation — password resets and weak hashes aid privesc
- [Linux Administration & Server Hardening](../Readme.md) — course hub
