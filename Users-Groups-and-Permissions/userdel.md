# userdel

## Overview

The `userdel` command removes user accounts from a Linux system. It deletes the account's records from `/etc/passwd`, `/etc/shadow`, `/etc/group`, and `/etc/gshadow`. By default it leaves the user's home directory and mail spool in place; those must be removed explicitly with `-r`. `userdel` is the low-level tool (from the `shadow-utils` package); on Debian/Ubuntu the higher-level `deluser` wrapper adds extra safety and cleanup.

> [!IMPORTANT]
> Deleting an account does not remove files the user owns elsewhere on the filesystem, nor does it terminate that user's running processes (unless `-f` is used). Plan for orphaned files and stale UID ownership after deletion.

## Commands

### Verify User Account Entries

- Display the last 3 lines of important account-related files along with file names:

```bash
tail -v -n 3 /etc/passwd /etc/shadow /etc/group /etc/gshadow
```

### Delete a User

- Delete user `u19` without removing their home directory or mail spool:

```bash
userdel u19
```

- **Effects:**

	- Removes the user's entry from `/etc/passwd`

	- Removes the user's password entry from `/etc/shadow`

	- Removes the user's group entries if applicable

	- Does **not** remove the user's home directory

	- Does **not** remove the user's mail spool

### Delete a User and Remove Home Directory

- Delete user `u1` and remove their home directory and mail spool:

```bash
userdel -r u1
```

- **Effects:**

	- Deletes the user account

	- Removes the user's home directory

	- Removes the user's mail spool (for example, `/var/mail/u1`)

### Force Delete a User

- Forcefully delete user `u16` and remove their home directory and mail spool:

```bash
userdel -f -r u16
```

- **Effects:**

	- Forces account removal even if the user is currently logged in

	- Removes the user's home directory

	- Removes the user's mail spool

	- May leave running processes owned by the deleted UID

> [!WARNING]
> Use `-f` with caution. It can leave orphaned files and processes on the system.

### Display User Deletion Help

```bash
userdel --help
```

## Verification

### Verify User Removal

- Check whether the user still exists:

```bash
id u19
```

- Search for the user in `/etc/passwd`:

```bash
grep '^u19:' /etc/passwd
```

- Search for the user in `/etc/shadow`:

```bash
grep '^u19:' /etc/shadow
```

- Check whether the home directory still exists:

```bash
ls -ld /home/u19
```

### Find Files Owned by a Deleted User

- Search for files owned by a specific UID:

```bash
find / -uid 1015 2>/dev/null
```

> Replace `1015` with the UID previously assigned to the deleted user.

## Best Practices

- Before deleting, capture the account's UID (`id <user>`) so you can locate its orphaned files afterward with `find / -uid <uid>`.
- Prefer **disabling** over deleting for accounts that may be needed for audit or forensic review: `usermod -L` plus a non-login shell keeps the record intact.
- Archive the home directory (`tar`) before using `-r` if the data has any retention or evidentiary value.
- Terminate the user's sessions and processes (`pkill -u <user>`) first, then delete, to avoid the `-f` orphaned-process situation.

## Security Considerations

- Files left behind after deletion retain the old numeric UID. If that UID is later reused for a new account, the new user silently inherits ownership of those files — a real access-control pitfall. Always sweep for orphaned files after deletion.
- Departing-employee and terminated-account cleanup is a common audit finding; stale or never-removed accounts are a standing access risk (CIS access-control guidance).
- `-f` may leave running processes under the deleted UID that continue to hold resources or network sockets; verify with `ps -u <uid>` / `pkill -u`.

## Troubleshooting

| Symptom | Likely cause / fix |
| :-- | :-- |
| `userdel: user is currently logged in` | active session/process; log the user out or use `-f` |
| Home directory remains after deletion | `-r` was omitted; remove it manually or re-run planning with `-r` |
| Orphaned files show a bare number for owner | UID no longer maps to a name; locate them with `find / -uid <uid>` |
| Group not removed | the account's primary group still has other members or is a shared group |

## Related

- [User-and-Group-Management](User-and-Group-Management.md) — parent topic
- [deluser](deluser.md) — higher-level user removal wrapper
- [usermod](usermod.md) — modify (or disable) rather than delete users
- [User-Management-with-useradd-and-adduser](User-Management-with-useradd-and-adduser.md) — inverse: creating users
- [Passwd-File-Linux-User-Account-File](Passwd-File-Linux-User-Account-File.md) — the record `userdel` removes
- [Linux Administration & Server Hardening](../Readme.md) — course hub
