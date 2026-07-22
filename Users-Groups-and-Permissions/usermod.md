# usermod

## Overview

The `usermod` command modifies existing user accounts in Linux. It edits the account's records in `/etc/passwd` and `/etc/shadow`: the comment (GECOS) field, login shell, UID, primary and supplementary groups, home directory, password hash, lock state, and account/password expiry. It is the counterpart to `useradd` (create) and `userdel` (remove), and one of the most security-relevant account tools because it can change UIDs and group membership — including granting administrative access.

> [!IMPORTANT]
> Many `usermod` changes only take effect on the user's *next login/session*. If the account is currently logged in, some options warn or refuse. Verify results with `id`, `groups`, and `chage -l` after each change.

## Options Reference

| Option | Purpose |
| :-- | :-- |
| `-c` | Set the comment/GECOS field |
| `-s` | Set the login shell |
| `-u` | Set the numeric UID |
| `-o` | Allow a non-unique (duplicate) UID (with `-u`) |
| `-g` | Set the primary group |
| `-G` | Set (replace) the supplementary group list |
| `-aG` | Append supplementary groups (use *with* `-G`) |
| `-d` | Set the home directory |
| `-m` | Move home directory contents (with `-d`) |
| `-p` | Set an already-encrypted password hash |
| `-L` / `-U` | Lock / unlock the account password |
| `-e` | Set the account expiration date |
| `-f` | Set the post-expiry inactivity period |

## Commands

## Display Help

```bash
usermod --help
```

### Change User Comment (GECOS Field)

```bash
usermod -c "HR user" u19
```

### Change Login Shell

- Set shell to `/bin/sh`:

```bash
usermod -s /bin/sh u19
```

- Disable interactive login:

```bash
usermod -s /sbin/nologin u19
```

> On some distributions the path may be `/usr/sbin/nologin`.

### Change UID

```bash
usermod -u 1030 u19
```

### Change Primary Group

- Using GID:

```bash
usermod -g 1027 u19
```

- Using group name:

```bash
usermod -g armour u19
```

## Change Home Directory

- Change home directory path only:

```bash
usermod -d /data/u19 u19
```

- Change home directory and move existing files:

```bash
usermod -m -d /opt/u19 u19
```

### Set Password Using Encrypted Hash

Generate password hashes:

- MD5 Hash

```bash
openssl passwd -1 123
```

- SHA-512 Hash

```bash
openssl passwd -6 123
```

- Set Password Using Hash

```bash
usermod -p '$1$1ZTV4fwe$4JQsRD3OR1/ol.1B5G3bj0' u19
```

```bash
usermod -p '$6$cIn5QyuN4TE5Dirg$AkruYKGDAwBn3ELyM7Vt2KSDCapZkN4JQwi1857HUwz6elY.kvdKs51ULbCoKwibD1EiMnCZZ0TtO.mgl0Hbb1' u15
```

> `openssl passwd 123` does not generate a DES hash on modern systems by default. Use `-1`, `-5`, or `-6` to explicitly select the hashing algorithm.

> [!TIP]
> Single-quote the hash passed to `-p` so the shell does not expand `$` sequences. Prefer SHA-512 (`-6`) or the system default over the legacy MD5 (`-1`) shown here.

### Lock and Unlock Account

- Lock account:

```bash
usermod -L u19
```

- Unlock account:

```bash
usermod -U u19
```

### Set Account Expiration Date

- Expire account on December 30, 2020:

```bash
usermod -e 2020-12-30 u19
```

- Remove account expiration:

```bash
usermod -e "" u19
```

### Set Password Inactivity Period

- Disable account 8 days after password expiration:

```bash
usermod -f 8 u19
```

- Disable inactivity feature:

```bash
usermod -f -1 u19
```

### Use Non-Unique UID

- Assign UID 0 to `u19`:

```bash
usermod -o -u 0 u19
```

> [!WARNING]
> This effectively gives the account root-level privileges and should be avoided except in very specific administrative scenarios. A non-`root` account with UID 0 is a classic backdoor/persistence technique — flag any duplicate UID-0 account during an audit.

### Manage Supplementary Groups

- Replace all supplementary groups:

```bash
usermod -G admin,admin1 u19
```

- Append groups without removing existing memberships:

```bash
usermod -aG admin,admin1 u19
```

> [!WARNING]
> `-G` *without* `-a` **replaces** the entire supplementary group list — any group not listed is removed. Use `-aG` to add groups while preserving existing memberships.

### Password Management

- Change current user's password:

```bash
passwd
```

- Change another user's password:

```bash
passwd u19
```

### Group Examples

- Set primary group:

```bash
usermod -g armour u19
```

- Set supplementary group list:

```bash
usermod -G IT u19
```

- Replace all supplementary groups:

```bash
usermod -G u1,u2,EMP u19
```

- Append additional groups:

```bash
usermod -aG u1,u2,EMP u19
```

#### Important Note About `-aG`

- The following command adds user `u18` to group `u17`:

```bash
usermod -aG u17 u18
```

- This works only if `u17` is a valid group name. The syntax is:

```bash
usermod -aG <group> <user>
```

## Verification

### Useful Verification Commands

- Check user details:

```bash
id u19
```

- View account entry:

```bash
grep '^u19:' /etc/passwd
```

- View shadow entry:

```bash
grep '^u19:' /etc/shadow
```

- View group memberships:

```bash
groups u19
```

- View password aging and account information:

```bash
chage -l u19
```

## Best Practices

- Use `-aG` (append) for group membership changes; reserve bare `-G` for the rare case where you deliberately want to reset the whole list.
- When changing a UID, follow up by re-owning the user's files (`find / -uid <old-uid> -exec chown <user> {} +`) so they are not left orphaned.
- Disable service and departing-user accounts with `usermod -L` plus `-s /usr/sbin/nologin` rather than deleting them, preserving the audit record.
- Set account expiry (`-e`) for temporary or contractor accounts so access self-terminates.

## Security Considerations

- **UID 0 / `-o`**: any account with UID 0 is root-equivalent. `usermod -o -u 0` on a non-root user is a well-known persistence/backdoor and should never exist on a hardened system (CIS: unique UIDs, single UID-0 account).
- **Sensitive groups**: adding a user to `sudo`, `wheel`, `adm`, `docker`, `lxd`, or `disk` via `-aG` frequently equals a path to root — treat these membership changes as privilege grants.
- **`-p` hashes**: the hash appears in the process list and shell history at set time; injecting a known hash is an offensive credential-implant technique. Prefer interactive `passwd`.
- Log and review `usermod` invocations; unexpected group additions or UID changes are high-value indicators of compromise.

## Troubleshooting

| Symptom | Likely cause / fix |
| :-- | :-- |
| `usermod: user u19 is currently logged in` | some changes are refused while logged in; log the user out and retry |
| Group change did not take effect | supplementary group changes apply on next login; re-login and check `id` |
| Home files missing after `-d` | `-m` was omitted, so files were not moved; move them or re-run with `-m` |
| Set password does not authenticate | hash not single-quoted, or algorithm unsupported by the PAM config |
| Files show a numeric owner after UID change | old-UID files were not re-owned; fix with `find / -uid <old-uid> -exec chown ...` |

## Related

- [User-and-Group-Management](User-and-Group-Management.md) — parent topic
- [userdel](userdel.md) — remove users
- [passwd](passwd.md) — change account passwords
- [gpasswd](gpasswd.md) — manage group membership and group passwords
- [User-Management-with-useradd-and-adduser](User-Management-with-useradd-and-adduser.md) — create the accounts you modify
- Privilege-Escalation — adding users to sudo/admin groups or UID 0 is privesc
- [Linux Administration & Server Hardening](../Readme.md) — course hub
