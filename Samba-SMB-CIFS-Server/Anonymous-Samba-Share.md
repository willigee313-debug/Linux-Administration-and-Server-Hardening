# Anonymous Samba Share

## Overview

Anonymous (guest) shares allow users to access shared directories **without authentication**. This is useful for public, read/write drop areas within a trusted LAN — for example a shared scratch space or a kiosk drop folder. Because there is no login, every operation is mapped to a single low-privilege account (`nobody`), and the share must never be exposed to untrusted networks.

> [!WARNING]
> Anonymous shares grant access to anyone who can reach TCP 445/139. Only deploy them on trusted, segmented networks — never on internet-facing systems or where the data is sensitive.

## Concepts

| Directive | Effect |
| --- | --- |
| `guest ok = yes` | Allows connections without a username/password. |
| `guest only = yes` | Forces *every* connection to be treated as a guest. |
| `force user = nobody` | Runs all filesystem operations as the unprivileged `nobody` user. |
| `create mode` / `directory mode` | POSIX mode applied to new files/directories (here `0777`). |
| `writable = yes` / `read only = no` | Permits write and delete operations. |

## Prepare Public Directory

Create a public directory with full access permissions:

```bash
mkdir /Public
```

Set open permissions for read, write, and execute access:

```bash
chmod 777 /Public/
```

Or recursively apply to all contents inside:

```bash
chmod -R 777 /Public/
```

Set ownership to the `nobody` user and group:

```bash
chown -R nobody:nobody /Public/
```

Verify the directory exists and has correct permissions:

```bash
ls -lh / | grep Public
```

## Configure Samba For Anonymous Access

Edit the Samba configuration file:

```bash
vim /etc/samba/smb.conf
```

Append or modify the following share definition:

```ini
[Public]
    path = /Public
    writable = yes
    guest ok = yes
    guest only = yes
    read only = no
    create mode = 0777
    directory mode = 0777
    force user = nobody
```

This configuration:

- Enables guest-only access.
- Allows full read/write/delete capabilities.
- Forces all operations as the `nobody` user.

## Restart Samba Service

Apply the configuration by restarting the Samba service:

```bash
systemctl restart smb.service
```

## Test Anonymous Share Access

Check the list of available shares on the Samba server:

```bash
smbclient -L 192.168.1.33
```

Try accessing the public share anonymously:

```bash
smbclient //192.168.1.33/Public/
```

Explicitly use guest access with no credentials:

```bash
smbclient -L 192.168.1.33 -U "" -N
```

Or connect directly to the public share as a guest:

```bash
smbclient //192.168.1.33/Public -U "" -N
```

Another equivalent form:

```bash
smbclient -U "" //192.168.1.33/Public -N
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: smbclient listing showing the Public share available for anonymous guest access alongside the default IPC$ share_

## Additional Sample Configuration

Here's an extended `smb.conf` snippet with additional shares:

```bash
vim /etc/samba/smb.conf
```

```ini
# See smb.conf.example for a more detailed config file or
# read the smb.conf manpage.
# Run 'testparm' to verify the config is correct after
# you modified it.
# Anonymous Samba Share
# Note:
# SMB1 is disabled by default. This means clients without support for SMB2 or
# SMB3 are no longer able to connect to smbd (by default).

[global]
	workgroup = SAMBA
	security = user

	passdb backend = tdbsam

	printing = cups
	printcap name = cups
	load printers = yes
	cups options = raw

	# Install samba-usershares package for support
	include = /etc/samba/usershares.conf

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
	# printadmin is a local group
	write list = printadmin root
	force group = printadmin
	create mask = 0664
	directory mask = 0775

[data]
    comment = Data
    path = /opt/data
    public = yes
    writable = yes
    guest ok = no
    guest only = no

[backup]
    comment = Server Backup
    path = /backup
    public = yes
    writable = yes
    valid users = infosec, armour

[Public]
    path = /Public
    writable = yes
    guest ok = yes
    guest only = yes
    read only = no
    create mode = 0777
    directory mode = 0777
    force user = nobody
```

> [!IMPORTANT]
> Make sure the directories `/opt/data` and `/backup` exist with proper ownership and permissions for the listed users.

```bash
systemctl restart smb.service
```

## Security Considerations

- Anonymous shares should only be used in **trusted networks**. Avoid enabling guest access on internet-facing systems or in environments where data sensitivity is a concern.
- A `0777` directory is world-writable — scope it to a dedicated volume and monitor its contents; it can become a dumping ground for malware or unauthorized data.
- Keep guest access confined to the single `[Public]` share; do not add `guest ok = yes` to shares holding real data.
- Ensure SMB1 remains disabled (default) so only SMB2/SMB3 clients connect.

## Related Notes

- [Samba-Server-Setup](Samba-Server-Setup.md) — base Samba server setup
- [Samba-SMB-CIFS-Server](Samba-SMB-CIFS-Server.md) — Samba/SMB/CIFS module hub
- [Share-With-Selected-Users](Share-With-Selected-Users.md) — authenticated-share counterpart
- [Shared-Common-Directories-With-Samba](Shared-Common-Directories-With-Samba.md) — related share configuration
- [Smb-Client-tools](Smb-Client-tools.md) — client tools for SMB shares
- SMB-Enumeration — attacker-side SMB recon
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
