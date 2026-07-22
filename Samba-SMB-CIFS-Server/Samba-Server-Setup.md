# Samba Server Setup

## Overview

This guide outlines how to install, configure, manage, and test a Samba server on **RHEL / CentOS**-based systems using `yum` and related tools. Samba lets Linux hosts share files and printers with Windows clients over the SMB/CIFS protocol. It walks the full lifecycle: package install, verification, `smb.conf` configuration, user management, service control, firewall rules, client testing, and Samba's backing TDB databases.

> [!NOTE]
> Commands here use `yum` and `firewalld`, targeting RHEL/CentOS/Rocky/AlmaLinux. On Debian/Ubuntu the package is installed with `apt install samba` and the firewall is managed with `ufw`.

## Installation

Install the base Samba package:

```bash
yum install samba
```

Install all related Samba packages (clients, tools, etc.):

```bash
yum install samba*
```

## Verifying Samba Installation

Once installed, RPM query flags let you inspect what was placed on disk.

| Command | Purpose |
| --- | --- |
| `rpm -qa \| grep samba` | List installed Samba packages. |
| `rpm -qi samba` | Detailed info on the Samba package. |
| `rpm -qc samba` | List Samba configuration files. |
| `rpm -qd samba` | List Samba documentation files. |
| `rpm -ql samba` | List all files installed by the package. |

List installed Samba packages:

```bash
rpm -qa | grep samba
```

Get detailed info on the Samba package:

```bash
rpm -qi samba
```

List Samba configuration files:

```bash
rpm -qc samba
```

List Samba documentation files:

```bash
rpm -qd samba
```

List all files installed by the Samba package:

```bash
rpm -ql samba
```

## Configuration

Open the main Samba configuration file using a text editor:

```bash
vim /etc/samba/smb.conf
```

> [!TIP]
> Refer to `/etc/samba/smb.conf.example` or `man smb.conf` for detailed syntax, and run `testparm` to validate your file before restarting the service.

### Example Minimal Configuration

```bash
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
```

## Managing Samba Users

> [!IMPORTANT]
> Every Samba user must first exist as a **Linux system user**. The `smbpasswd` database only stores the SMB credential; the underlying account, UID, and home directory come from the system.

Create a Linux user:

```bash
useradd smb-user1
```

```bash
useradd infosec
```

View the `smbpasswd` command help:

```bash
smbpasswd -h
```

Or:

```bash
smbpasswd --help
```

Add users to Samba (each must first exist as a Linux user):

```bash
smbpasswd -a smb-user1
```

```bash
smbpasswd -a armour
```

```bash
smbpasswd -a infosec
```

## Managing Samba Services

Check the Samba service status:

```bash
systemctl status smb.service
```

Start the Samba service:

```bash
systemctl start smb.service
```

Enable the Samba service on boot:

```bash
systemctl enable smb.service
```

## Networking

Check listening SMB ports:

```bash
netstat -nltup
```

Filter for Samba-related ports:

```bash
netstat -nltup | grep smb
```

## Firewall Configuration With Firewalld

> [!NOTE]
> Samba uses ports **137/udp**, **138/udp**, **139/tcp**, and **445/tcp**.

| Port | Protocol | Service |
| --- | --- | --- |
| 137 | UDP | NetBIOS Name Service |
| 138 | UDP | NetBIOS Datagram Service |
| 139 | TCP | NetBIOS Session Service (SMB over NetBIOS) |
| 445 | TCP | SMB directly over TCP (modern SMB2/SMB3) |

Start and enable firewalld:

```bash
systemctl start firewalld
```

```bash
systemctl enable firewalld
```

Check the firewall state:

```bash
firewall-cmd --state
```

Add the Samba service to the firewall (permanent):

```bash
firewall-cmd --permanent --add-service=samba
```

You can also add individual ports if preferred:

```bash
firewall-cmd --permanent --add-port=137/udp
firewall-cmd --permanent --add-port=138/udp
firewall-cmd --permanent --add-port=139/tcp
firewall-cmd --permanent --add-port=445/tcp
```

Reload the firewall to apply changes:

```bash
firewall-cmd --reload
```

Verify active rules:

```bash
firewall-cmd --list-all
```

## Testing Samba Shares From A Client

List available shares (anonymous):

```bash
smbclient -L 192.168.1.33
```

List shares with user authentication:

```bash
smbclient -L 192.168.1.33 -U armour
```

```bash
smbclient -L 192.168.1.33 -U infosec
```

Connect to a share:

```bash
smbclient //192.168.1.33/armour -U armour
```

```bash
smbclient //192.168.1.33/infosec -U infosec
```

### Working Inside The smbclient Shell

Once connected, `smbclient` presents an FTP-like `smb:\>` prompt.

| Command | Action |
| --- | --- |
| `ls` | List files and directories. |
| `cd <foldername>` | Change directory. |
| `get <remote_filename>` | Download a file. |
| `put <local_filename>` | Upload a file. |
| `mget *.txt` | Download multiple files. |
| `mput *.pdf` | Upload multiple files. |
| `prompt` | Toggle per-file confirmation for `mget`/`mput`. |
| `mkdir <dirname>` | Create a directory. |
| `del <filename>` | Remove a file. |
| `rmdir <dirname>` | Remove a directory. |
| `cd ..` / `cd /` | Move up / to the share root. |
| `exit` | Leave the SMB shell. |

List files and directories:

```bash
ls
```

Change directory:

```bash
cd <foldername>
```

Example:

```bash
cd reports
```

Download a file from the share:

```bash
get <remote_filename>
```

Example:

```bash
get confidential.txt
```

Upload a file to the share:

```bash
put <local_filename>
```

Example:

```bash
put notes.txt
```

Download multiple files:

```bash
mget *.txt
```

You will be prompted for each file. Use `prompt` to toggle confirmation:

```bash
prompt
```

Upload multiple files:

```bash
mput *.pdf
```

Go back to the previous directory:

```bash
cd ..
```

Go to the root of the share:

```bash
cd /
```

Create a new directory:

```bash
mkdir <dirname>
```

Example:

```bash
mkdir backups
```

Remove a file:

```bash
del <filename>
```

Remove a directory:

```bash
rmdir <dirname>
```

Exit the SMB shell:

```bash
exit
```

## Samba Password And TDB Database Management

Navigate to the Samba private database directory:

```bash
cd /var/lib/samba/private
```

Files present:

```text
passdb.tdb  secrets.ldb  secrets.tdb
```

List Samba user info verbosely:

```bash
pdbedit -L -v
```

List Samba users in `smbpasswd` format:

```bash
pdbedit -L -w
```

### Understanding Samba's Database Files

| File | Purpose | Location |
| --- | --- | --- |
| `passdb.tdb` | Samba user account info (passwords, flags). Used when `passdb backend = tdbsam`. | `/var/lib/samba/private/` or `/var/lib/samba/` |
| `secrets.tdb` | Internal secrets: domain membership keys, trust passwords, LDAP bind credentials. Highly sensitive — root-only. | `/var/lib/samba/private/` |
| `secrets.ldb` | AD DC secrets: domain controller secrets, machine credentials, replication metadata. Part of Samba's LDB store. | `/var/lib/samba/private/` |

#### `passdb.tdb`

- **Purpose**: Stores Samba user account information (passwords, flags, etc.).
- **Backend**: Used when `passdb backend = tdbsam` is set in `smb.conf`.
- **Location**: Usually found in `/var/lib/samba/private/` or `/var/lib/samba/`.

#### `secrets.tdb`

- **Purpose**: Contains internal secrets used by Samba:
    - Domain membership keys
    - Trust passwords
    - LDAP bind credentials
- **Highly sensitive**: Should be readable by root only.
- **Location**: Typically in `/var/lib/samba/private/`

#### `secrets.ldb`

- **Purpose**: Used in Samba AD DC environments.
- **Part of**: Samba's LDB (LDAP-like) database when acting as an Active Directory domain controller.
- **Stores**: Domain controller secrets, machine credentials, replication metadata, etc.
- **Location**: Usually in `/var/lib/samba/private/`

> [!NOTE]
> If you're using Samba as an AD DC (with `samba-tool domain provision`), `secrets.ldb` replaces or supplements `secrets.tdb`.

View contents (advanced tool required):

```bash
ldbsearch -H /var/lib/samba/private/secrets.ldb
```

> [!WARNING]
> Do **not** edit these files manually. Always back them up before upgrading or migrating Samba.

Restrict permissions:

```bash
chmod 600 /var/lib/samba/private/*
```

```bash
chown root:root /var/lib/samba/private/*
```

## TDB Tools For Database Management

Install TDB tools:

```bash
yum install tdb-tools
```

```bash
apt install tdb-tools
```

Verify the TDB tools installation:

```bash
rpm -qa | grep tdb-tools
```

```bash
rpm -qi tdb-tools
```

```bash
rpm -ql tdb-tools
```

Dump Samba TDB databases:

```bash
tdbdump /var/lib/samba/private/passdb.tdb
```

```bash
tdbdump /var/lib/samba/private/secrets.tdb
```

Or from the current working directory:

```bash
tdbdump passdb.tdb
```

```bash
tdbdump secrets.tdb
```

## Best Practices

- Validate configuration with `testparm` before every service restart.
- Keep SMB1 disabled; require SMB2/SMB3 clients.
- Give each Samba share the tightest `valid users` / `write list` that still works.
- Back up `passdb.tdb`, `secrets.tdb`, and `secrets.ldb` before upgrades or migrations.
- Restrict `/var/lib/samba/private/*` to `root:root` mode `600`.

## Troubleshooting

| Symptom | Likely cause / fix |
| --- | --- |
| Service won't start after config edit | Run `testparm`; fix the reported directive. |
| Ports not listening | Confirm `smb.service` is running and firewalld allows Samba (`firewall-cmd --list-all`). |
| Auth fails for a valid user | Ensure the user exists both as a Linux user and in `smbpasswd -a`. |
| Cannot browse shares from Windows | Verify ports 137–139/445 are open and NetBIOS/`nmbd` is running if needed. |

## Related Notes

- [Samba-SMB-CIFS-Server](Samba-SMB-CIFS-Server.md) — Samba/SMB/CIFS module hub
- [Smb-Client-tools](Smb-Client-tools.md) — client tools for SMB shares
- [Anonymous-Samba-Share](Anonymous-Samba-Share.md) — guest/anonymous share configuration
- [Share-With-Selected-Users](Share-With-Selected-Users.md) — authenticated share configuration
- [Shared-Common-Directories-With-Samba](Shared-Common-Directories-With-Samba.md) — sharing common directories
- SMB-Enumeration — attacker-side SMB recon
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
