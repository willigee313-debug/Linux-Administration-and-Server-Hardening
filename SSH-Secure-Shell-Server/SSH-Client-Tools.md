# SSH Client Tools

## Overview

SSH (Secure Shell) is a cryptographic network protocol that allows secure access to remote systems over unsecured networks. It encrypts the entire session, including passwords and commands. Beyond the interactive `ssh` client itself, the OpenSSH client suite ships a family of tools that all ride the same encrypted transport: `scp` and `rsync` for file transfer, `ssh-agent`/`ssh-add` for key management, plus Windows equivalents like PuTTY and WinSCP.

This note is a practical command reference for the client side of SSH — connecting, forwarding ports, jumping through bastion hosts, copying files, synchronising directories, and configuring convenient aliases.

## Concepts

The client tools and what each is for:

| Tool | Purpose |
| --- | --- |
| `ssh` | Interactive login, remote command execution, port forwarding, proxy/jump |
| `scp` | Simple secure file copy over SSH |
| `rsync` | Efficient, resumable, incremental file/directory synchronisation over SSH |
| `ssh-agent` / `ssh-add` | Hold decrypted private keys in memory to avoid repeated passphrase prompts |
| PuTTY / PuTTYgen | Windows SSH client and key generator (`.ppk` format) |
| WinSCP | Windows GUI file transfer (SFTP/SCP) |

## SSH Client Tool

### Basic SSH Usage

- Connect to a remote system using the default username and port 22.

```bash
ssh 192.168.1.33
```

- Use the `-l` flag or direct format to log in as the `root` user.

```bash
ssh -l root 192.168.1.33
```

```bash
ssh root@192.168.1.33
```

- Connect using a non-default port when the SSH server is not on port 22.

```bash
ssh -l root -p 2222 192.168.1.33
```

```bash
ssh -p 2200 root@192.168.1.33
```

- Specify a private key for authentication, bypassing password prompts.

```bash
ssh -i id_rsa_centos root@192.168.1.33
```

```bash
ssh -i ~/.ssh/id_ecdsa root@192.168.1.33
```

```bash
ssh -p 2200 -i ~/.ssh/id_ecdsa root@192.168.1.33
```

- Get more detailed connection output to debug issues.

```bash
ssh -v root@192.168.1.33
```

```bash
ssh -vvv -i ~/.ssh/id_rsa root@192.168.1.33
```

> [!TIP]
> Increase verbosity by stacking `-v` (`-v`, `-vv`, `-vvv`). It reveals which key was offered, which auth method the server accepted, and where a handshake fails — the first thing to reach for when a connection is refused.

- Run one-line commands on the remote machine without logging in interactively.

```bash
ssh root@192.168.1.33 id
```

```bash
ssh root@192.168.1.33 -p 22 id
```

```bash
ssh root@192.168.1.33 'uptime'
```

```bash
ssh root@192.168.1.33 -p 22 -i .ssh/id_rsa 'cat /etc/hostname'
```

```bash
ssh -i ~/.ssh/id_rsa root@192.168.1.33 "df -h"
```

Common `ssh` flags used above:

| Flag | Meaning |
| --- | --- |
| `-l <user>` | Login username (alternative to `user@host`) |
| `-p <port>` | Connect to a non-default port |
| `-i <keyfile>` | Use a specific private key (identity file) |
| `-v` / `-vvv` | Verbose / very verbose debugging output |
| `-L` | Local port forward |
| `-R` | Remote port forward |
| `-J` | Jump host (ProxyJump) |

### Port Forwarding

- Forward a local port to a port on the remote server.

```bash
ssh -L 8080:localhost:80 root@192.168.1.33
```

- Forward a remote port to a port on the local system.

```bash
ssh -R 9090:localhost:80 root@192.168.1.33
```

### Use Jump Host (SSH Proxying)

- Connect through an intermediary jump host (gateway) to reach a private/internal system.

```bash
ssh -J root@gateway root@192.168.1.33
```

```mermaid
flowchart LR
    you["Your workstation"] -->|"SSH"| gw["Jump host / bastion\n(gateway)"]
    gw -->|"SSH (internal)"| target["Internal target\n192.168.1.33"]
```

## Secure Copy (`scp`)

`scp` uses SSH to transfer files securely between systems.

- Upload a file to a remote server's directory.

```bash
scp file.txt root@192.168.1.33:/root/
```

```bash
scp ftp-server.py root@192.168.1.33:/tmp/
```

- Transfer several files at once.

```bash
scp file1.txt file2.txt root@192.168.1.33:/root/
```

```bash
scp image.png notes.md root@192.168.1.33:/var/www/html/
```

```bash
scp *.zip root@192.168.1.33:/root/
```

- Use `-r` to copy directories and their contents.

```bash
scp -r /local/dir root@192.168.1.33:/remote/dir
```

```bash
scp -r ~/Documents root@192.168.1.33:/tmp/
```

- Download files from a remote machine to your local system.

```bash
scp -r root@192.168.1.33:/root/file.txt /local/dir/
```

```bash
scp -r root@192.168.1.33:/var/www/html/ /tmp/
```

- Limit bandwidth to avoid saturating the network.

```bash
scp -l 500 file.zip root@192.168.1.33:/tmp/
```

## Rsync Commands

`rsync` is a fast, reliable tool for copying and synchronizing files and directories between local and remote systems. It uses SSH for secure transfers and supports options for incremental syncing, compression, exclusion, bandwidth limits, and more.

```bash
apt install rsync
```

```bash
yum install rsync
```

Frequently used `rsync` options referenced below:

| Option | Effect |
| --- | --- |
| `-a` | Archive mode (preserves permissions, timestamps, ownership, symlinks) |
| `-v` | Verbose progress output |
| `-z` | Compress data in transit (faster over slow links) |
| `-l` | Preserve symbolic links as links |
| `-c` | Compare by checksum instead of size/mtime |
| `-n` / `--dry-run` | Simulate the transfer, change nothing |
| `--delete` | Remove destination files absent from the source (exact mirror) |
| `--progress` | Show per-file transfer progress |
| `--partial` | Keep partially transferred files to allow resume |
| `--bwlimit=<KB/s>` | Cap bandwidth usage |
| `--include` / `--exclude` | Filter which files are transferred |
| `--ignore-existing` | Skip files already present at the destination |
| `--remove-source-files` | Delete source files after successful copy (move) |
| `--itemize-changes` | Detailed per-item change listing |
| `--log-file=<path>` | Write a transfer log for auditing |

- Copy files or directories from a remote server to your local machine.

```bash
rsync -av root@192.168.1.33:/var/log /tmp/
```

```bash
rsync -az root@192.168.1.33:/var/log /tmp/
```

> `-a`: Archive mode (preserves permissions, timestamps, etc.)
> `-v`: Verbose (shows detailed progress)
> `-z`: Compress data during transfer (faster over slow connections)

- Upload local data to a remote machine.

```bash
rsync -av /tmp/log root@192.168.1.33:/var/
```

```bash
rsync -az /tmp/log root@192.168.1.33:/var/
```

- Specify a private key to connect securely.

```bash
rsync -av -e 'ssh -i ~/.ssh/id_rsa' root@192.168.1.33:/var/log/ /tmp/log
```

- Connect through a non-default SSH port.

```bash
rsync -av -e 'ssh -p 2200' /backup root@192.168.1.100:/mnt/backup
```

- Preview changes without making them.

```bash
rsync -av --dry-run root@192.168.1.33:/var/log /tmp/
```

> **`--dry-run`**: Simulates the transfer, allowing you to preview what will happen without making any actual changes.

- Mirror local directory exactly.

```bash
rsync -av --delete root@192.168.1.33:/var/log /tmp/
```

> **`--delete`**: This option removes files in the destination directory (`/tmp/` in this case) that no longer exist in the source directory (`root@192.168.1.33:/var/log`).
> **`root@192.168.1.33:/var/log`**: This specifies the source directory you are syncing from. It’s on the remote machine with the IP `192.168.1.33`, and the user is `root`.
> **`/tmp/`**: This is the destination directory on your local machine where files from the source (`/var/log`) will be copied to.

> [!WARNING]
> `--delete` makes the destination an exact mirror — files present only at the destination are **permanently removed**. Always run with `--dry-run` first to preview the deletions.

- Show Transfer Progress

```bash
rsync -av --progress largefile.iso root@192.168.1.33:/iso/
```

> **`--progress`**: This option shows the progress of the transfer, including the percentage of completion, transfer speed, and estimated time remaining for each file.
> **`largefile.iso`**: This is the file you are transferring. It is assumed to be located in the current directory of your local machine.
> **`root@192.168.1.33:/iso/`**: This is the destination directory on the remote system. The file will be transferred to the `/iso/` directory on the remote machine at IP address `192.168.1.33`, using the `root` user.

- Sync Only Specific File Types

```bash
rsync -av --include='*.jpg' --exclude='*' /photos/ root@192.168.1.33:/backup/images/
```

> **`--include='*.jpg'`**: This option tells `rsync` to include files with the `.jpg` extension.
> **`--exclude='*'`**: This option tells `rsync` to exclude all files, except for those that match the `--include` pattern. In other words, all files are excluded unless they are `.jpg` files.
> **`/photos/`**: This is the source directory on your local machine where the files are located.
> **`root@192.168.1.33:/backup/images/`**: This is the destination directory on the remote system. The files will be transferred to `/backup/images/` on the machine with IP `192.168.1.33`.

- Exclude Files Or Directories

```bash
rsync -av --exclude 'node_modules' ~/project root@192.168.1.33:/deploy/
```

```bash
rsync -av --exclude '*.log' --exclude 'tmp/' /var/www/ root@192.168.1.33:/backup/
```

- Preserve Symlinks

```bash
rsync -av -l /var/www/ root@192.168.1.33:/www/
```

> **`-l`**: This option preserves symbolic links (i.e., copies the symlink itself rather than the file it points to). It ensures that symlinks are treated as symlinks during the transfer, rather than being followed or dereferenced.
> **`/var/www/`**: This is the source directory on your local machine.
> **`root@192.168.1.33:/www/`**: This is the destination directory on the remote system at `192.168.1.33`. The `root` user is used to perform the transfer.

- Resume interrupted transfers without restarting.

```bash
rsync -av --partial --progress bigfile.iso root@192.168.1.33:/iso/
```

> **`--partial`**: This option ensures that partially transferred files are kept if the transfer is interrupted. This way, if the transfer stops for any reason, you can resume from where it left off rather than starting over.  
> **`--progress`**: This option shows the progress of the transfer for large files, including the percentage completed, transfer speed, and estimated time remaining.
> **`bigfile.iso`**: This is the file you're transferring from your local machine.
> **`root@192.168.1.33:/iso/`**: This is the destination directory on the remote system (`192.168.1.33`), where the file will be copied to.

- Log file transfers for auditing.

```bash
rsync -av --log-file=/var/log/rsync-backup.log /data/ root@192.168.1.33:/backup/data/
```

> **`--log-file=/var/log/rsync-backup.log`**: This option writes the details of the file transfer to the specified log file (`/var/log/rsync-backup.log`). The log will include information like which files were transferred, skipped, or failed. This is useful for tracking the progress and results of the transfer.  
> **`/data/`**: This is the source directory on your local machine that you are syncing from.
> **`root@192.168.1.33:/backup/data/`**: This is the destination directory on the remote machine (`192.168.1.33`). The files will be copied to `/backup/data/`.

- Create dated folders during backups.

```bash
rsync -av /data/ root@192.168.1.33:/backup/$(date +%F)/
```

> **`/data/`**: This is the source directory on your local machine. The contents of this directory will be synced to the remote system.
> **`root@192.168.1.33:/backup/$(date +%F)/`**: This is the destination directory on the remote machine. The `$(date +%F)` part will be replaced by the current date in the `YYYY-MM-DD` format, so the backup will go to a folder named after the current date.

- Move files instead of copying.

```bash
rsync -av --remove-source-files /export/logs root@192.168.1.33:/logs/
```

> **`--remove-source-files`**: This option removes the files from the source directory (`/export/logs`) after they have been successfully copied to the destination (`/logs/` on the remote machine). **Note**: It only removes regular files, not directories.  
> **`/export/logs`**: The source directory on your local machine containing the files you want to transfer.  
> **`root@192.168.1.33:/logs/`**: The destination directory on the remote machine where the files will be copied.

- Limit Bandwidth

```bash
rsync -av --bwlimit=500 /iso root@192.168.1.33:/mnt/iso/
```

> **`--bwlimit=500`**: This option limits the bandwidth used by `rsync` to **500 KB/s**. You can adjust this value as needed (it is in kilobytes per second, not megabytes). This is useful if you're transferring large files but want to prevent the transfer from consuming all available network bandwidth.  
> **`/iso`**: This is the source directory on your local machine that contains the files you want to transfer.  
> **`root@192.168.1.33:/mnt/iso/`**: This is the destination directory on the remote system, where the files will be copied.

- Slower, but ensures integrity.

```bash
rsync -avc /important root@192.168.1.33:/verified_backup/
```

> **`-c`**: This option tells `rsync` to perform a **checksum** on the files to verify their integrity. By default, `rsync` compares files based on their size and timestamp, but with `-c`, it will instead check the file's content to ensure the files were transferred correctly.  
> **`/important`**: The source directory on your local machine. All files and subdirectories within this directory will be transferred.
> **`root@192.168.1.33:/verified_backup/`**: The destination directory on the remote system where the files will be copied. The files will be placed inside `/verified_backup/` on the remote machine.

- Don’t overwrite updated files.

```bash
rsync -av --ignore-existing /media/photos/ root@192.168.1.33:/album/
```

> **`--ignore-existing`**: This option ensures that **only new files** (files that do not already exist at the destination) will be copied to the remote system. Existing files in the destination directory will be skipped, even if they have changed.  
> **`/media/photos/`**: The source directory on your local machine, containing the files you want to copy.
> **`root@192.168.1.33:/album/`**: The destination directory on the remote machine, where files will be copied.

- Show Only Differences (No Transfer)

```bash
rsync -n -av --itemize-changes ~/projects root@192.168.1.33:/var/www/
```

> **`--itemize-changes`**: This option shows a detailed list of changes that would occur. It uses a special format to show exactly what would be transferred, what would be skipped, and why.  
> **`~/projects`**: The source directory on your local machine (in this case, `~/projects`). The tilde (`~`) refers to your home directory, so this is the `projects` folder in your home directory.  
> **`root@192.168.1.33:/var/www/`**: The destination directory on the remote machine, where the files will be transferred.

## Windows SSH Tools

### PuTTY and PuTTYgen

- **PuTTY**: A Windows SSH client supporting SSH, Telnet, serial, and SCP connections.

- **PuTTYgen**: Tool to generate or convert SSH keys to the `.ppk` format used by PuTTY.

### WinSCP

- GUI-based SSH file transfer client for Windows.

- Supports SFTP and SCP protocols.

- Allows drag-and-drop transfers and uses `.ppk` key files.

### Start SSH Agent and Add Key

- Keeps private keys in memory to avoid retyping passphrases.

```bash
eval "$(ssh-agent -s)"
```

```bash
ssh-add ~/.ssh/id_rsa
```

### Use SSH Config For Aliases

- Create an SSH config file for convenient short commands.

```conf
Host devserver
    HostName 192.168.1.33
    User root
    Port 2222
    IdentityFile ~/.ssh/id_rsa
    LocalForward 8080 localhost:80  # Example port forwarding
    ProxyJump gateway.example.com  # Example jump host (proxy)
```

> Now you can connect easily with:

```bash
ssh devserver
```

> [!TIP]
> Per-user client config lives in `~/.ssh/config`. Defining a `Host` alias lets you fold username, port, key, forwards, and jump host into a single memorable name.

### Disable Host Key Checking (For Automation Only)

- Suppress host key prompts in automated or non-interactive environments (not secure).

```bash
ssh -o StrictHostKeyChecking=no root@192.168.1.100
```

> [!WARNING]
> `StrictHostKeyChecking=no` disables the defence against man-in-the-middle attacks — the client will silently trust any host key. Use it only inside trusted, ephemeral automation, never for interactive administration.

## SSH Security Tips

- Use public/private key authentication over passwords.

- Set `PermitRootLogin no` to prevent direct root logins.

- Limit access using `AllowUsers` and firewall rules.

- Enable `fail2ban` to ban IPs with repeated failed logins.

- Regularly update the OpenSSH server and client software.

- **Disable weak ciphers** and **use strong key exchange algorithms**:

    ```conf
    Ciphers aes128-ctr,aes192-ctr,aes256-ctr
    KexAlgorithms diffie-hellman-group14-sha1
    ```

## Troubleshooting

| Symptom | First step |
| --- | --- |
| Connection refused / hangs | `ssh -vvv …` to see where the handshake stops |
| Key not accepted | Confirm correct `-i` key, and server-side `authorized_keys` permissions |
| Host key changed warning | Verify the host is genuine, then update `~/.ssh/known_hosts` |
| `rsync` deletes unexpected files | You used `--delete`; re-run with `--dry-run` to inspect |
| Slow transfer over WAN | Add `-z` compression and/or `--bwlimit` |

## References

- `ssh(1)`, `scp(1)`, `ssh_config(5)`, `rsync(1)` manual pages.
- PuTTY — https://www.putty.org/
- WinSCP — https://winscp.net/

## Related
- [SSH(Secure-Shell)-Server](SSH(Secure-Shell)-Server.md) — the server these tools connect to
- [SSH-Keygen-Usage-and-SSH-Authentication-Setup](SSH-Keygen-Usage-and-SSH-Authentication-Setup.md) — generating keys for these tools
- [SSH-Public-and-Private-Key-Configuration](SSH-Public-and-Private-Key-Configuration.md) — key-based auth setup
- SSH-Enumeration — attacker-side SSH recon
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
