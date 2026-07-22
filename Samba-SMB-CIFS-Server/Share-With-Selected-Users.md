# Share With Selected Users

## Overview

This setup enables **controlled access** to a Samba share (`/backup`) by restricting it to a named set of users with the `valid users` directive. Unlike an anonymous share, every connection must authenticate, and only accounts explicitly listed can mount or browse the share. This is the standard pattern for team shares that hold real (non-public) data.

## Concepts

| Directive | Effect |
| --- | --- |
| `valid users = infosec, armour` | Only these accounts may access the share; all others are denied. |
| `security = user` | Each connection must authenticate as a real Samba user. |
| `writable = yes` | Permits write access for authorized users. |
| `public = yes` | Makes the share visible during browsing (access is still gated by `valid users`). |

> [!IMPORTANT]
> Each user must exist **both** as a Linux system user *and* as a Samba user (`smbpasswd -a`). Missing either half results in "access denied" even with the correct password.

## Configure Samba Share For Selected Users

Edit the Samba configuration:

```bash
vim /etc/samba/smb.conf
```

Add or modify the following configuration:

```ini
[global]
    workgroup = SAMBA
    security = user
    passdb backend = tdbsam

    printing = cups
    printcap name = cups
    load printers = yes
    cups options = raw

[homes]
    comment = Home Directories
    valid users = %S, %D%w%S
    browseable = No
    read only = No
    inherit acls = Yes

[printers]
    comment = All Printers
    path = /var/tmp
    printable = Yes
    create mask = 0600
    browseable = No

[print$]
    comment = Printer Drivers
    path = /var/lib/samba/drivers
    write list = @printadmin root
    force group = @printadmin
    create mask = 0664
    directory mask = 0775

[backup]
    comment = Server Backup
    path = /backup
    public = yes
    writable = yes
    valid users = infosec, armour
```

> [!NOTE]
> The `valid users` directive ensures only `infosec` and `armour` can access the `backup` share.

## Restart Samba Service

Apply the configuration changes:

```bash
systemctl restart smb.service
```

## Access The Restricted Share

Access as `thw` (should be denied if not listed in `valid users`):

```bash
smbclient //192.168.1.33/backup -U smb-user1
```

Access as allowed users:

```bash
smbclient -L 192.168.1.33 -U infosec
```

```bash
smbclient -L 192.168.1.33 -U armour
```

## Additional Notes

Make sure the users (`infosec`, `armour`) exist as both **Linux system users** and **Samba users**.

Add Samba users with:

```bash
smbpasswd -a infosec
```

```bash
smbpasswd -a armour
```

Ensure the shared directory `/backup` has proper ownership/permissions:

```bash
mkdir -p /backup
```

```bash
chmod -R 770 /backup
```

```bash
chown root:users /backup
```

> [!TIP]
> You can also create a custom group (e.g., `sambashare`) and add `infosec` and `armour` to it for easier permission management — then set `chown root:sambashare /backup` and manage membership instead of editing the share for each new user.

## Security Considerations

- Prefer `valid users` (an allow-list) over relying on filesystem permissions alone; it fails closed.
- Combine share-level `valid users` with tight POSIX permissions (`770`) so the OS also enforces access.
- Keep group membership as the single source of truth when a team grows — avoid long comma lists that drift out of date.
- Never set `guest ok = yes` on a share that holds real data.

## Related Notes

- [Anonymous-Samba-Share](Anonymous-Samba-Share.md) — open-share counterpart
- [Shared-Common-Directories-With-Samba](Shared-Common-Directories-With-Samba.md) — related share configuration
- [Samba-Server-Setup](Samba-Server-Setup.md) — base Samba server setup
- [Samba-SMB-CIFS-Server](Samba-SMB-CIFS-Server.md) — Samba/SMB/CIFS module hub
- [Smb-Client-tools](Smb-Client-tools.md) — client tools for SMB shares
- SMB-Enumeration — attacker-side SMB recon
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
