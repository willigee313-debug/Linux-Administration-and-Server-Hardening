# TFTP Client

## Overview

This note covers the **client side** of TFTP: transferring files to and from a TFTP server using the two standard Linux clients — the classic **`tftp`** and the more capable **`atftp`** (Advanced TFTP). Both speak plain **UDP port 69** with no authentication, so the emphasis here is on getting transfers to work reliably (correct transfer mode, permissions) and verifying them.

Use these clients to validate a freshly configured TFTP server, pull firmware or boot images, or push a device configuration back to a provisioning host.

> [!NOTE]
> **Client vs. server**
> This note assumes a working TFTP server is already listening on UDP/69. To stand one up, see [Trivial-File-Transfer-Protocol(TFTP)-Server](Trivial-File-Transfer-Protocol(TFTP)-Server.md).

## Concepts

### tftp vs. atftp

| | `tftp` (classic) | `atftp` (advanced) |
|---|---|---|
| Interface | Interactive prompt + one-liner (`-c`) | Command-line flags only |
| Options | Minimal (mode, get, put) | Block size, timeout, tsize, multicast |
| Scripting | Good with `-c` | Excellent (pure CLI) |
| Package | `tftp` | `atftp` |
| Best for | Quick manual tests | Automation and tuned transfers |

### Transfer modes

TFTP moves data in one of two modes, and choosing the wrong one corrupts files:

| Mode | Alias | Use for |
|---|---|---|
| `binary` | `octet` | Images, executables, archives, kernels — anything non-text (raw bytes preserved) |
| `netascii` | ASCII | Plain-text files where newline translation is desired |

> [!WARNING]
> **Always use binary mode for non-text files**
> If you transfer a binary file in the default `netascii` mode, newline conversions can corrupt it. Set `binary` mode before any image/executable/archive transfer.

## Installing TFTP Client Utilities

- Install the basic TFTP client:

```bash
yum install tftp
```

```bash
apt install tftp
```

- Install the Advanced TFTP client (`atftp`), which supports more features:

```bash
yum install atftp
```

```bash
apt install atftp
```

## Commands

The interactive `tftp` client presents a `tftp>` prompt. The commands below are the ones you will reach for most.

| Command | Description |
|---|---|
| `mode` | Show current transfer mode |
| `binary` | Set transfer mode to binary / octet |
| `verbose` | Enable verbose output |
| `timeout <sec>` | Set the timeout for responses |
| `connect` | Change or set a new TFTP server to use |
| `status` | Show current session settings |
| `get <file>` | Download a file |
| `put <file>` | Upload a file |
| `quit` | Exit the TFTP client |

## Using Interactive TFTP Client

- To enter interactive mode, simply run:

```bash
tftp <server-ip>
```

> Once inside the TFTP prompt (`tftp>`), you can use various commands. Here are the two you mentioned:

- Setting Transfer Mode To Binary

```bash
binary
```

> This sets the transfer mode to **binary** (also called _octet mode_), which is required when transferring non-text files like images, executables, and archives.
> If you transfer a binary file in the default mode (`netascii`), the file can become corrupted due to newline conversions.
> Use `binary` mode to ensure raw bytes are preserved exactly.

- Checking Status

```bash
tftp> status
```

> This displays the current session settings, including:
> Transfer mode (e.g., `netascii` or `binary`)
> Verbose/debug status
> Timeout value
> Server IP/hostname (if already connected)


>Example output:

```text
Connected to 192.168.1.34.
Mode: binary
Timeout: 10 seconds
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: Interactive tftp session at the `tftp>` prompt showing the output of the `status` command with binary mode and a 10-second timeout_

## Examples

### Testing File Download (GET Operation)

- To test file downloading from a TFTP server using the classic `tftp` client:

```bash
tftp <server-ip>
```

```bash
tftp 192.168.1.34
```

> Example interaction:

```bash
get messages
```

```bash
quit
```

Or in a one-liner:

```bash
tftp -v <server-ip> -c get messages
```

```bash
tftp -v 192.168.1.34 -c get messages
```

- Downloading A File (GET Operation) using atftp

```bash
atftp -g -r messages -l messages 192.168.1.34
```

> `-g`: get (download) file from server
>`-r messages`: remote file name on the server
> `-l messages`: local file name to save as
>`192.168.1.34`: the TFTP server IP address

### Testing File Upload (PUT Operation)

- To upload a file to the TFTP server (write permission required on server):

```bash
tftp <server-ip>
```

```bash
tftp 192.168.1.34
```

> Example interaction:

```bash
put sample.txt
```

```bash
quit
```

> Or use a one-liner:

```bash
tftp -v <server-ip> -c put sample.txt
```

```bash
tftp -v 192.168.1.34 -c put sample.txt
```

- Uploading A File (PUT Operation) using atftp

```bash
atftp -p -r newfile.txt -l newfile.txt 192.168.1.34
```

> `-p`: put (upload) file to server
> `-r newfile.txt`: remote file name to write as on the server
> `-l newfile.txt`: local file to upload
> `192.168.1.34`: the TFTP server IP address

## Best Practices

- **Use `-v`/`--verbose` when diagnosing** — the extra output reveals block numbers, timeouts, and the exact error the server returned.

```bash
atftp --verbose -g -r test.txt -l test.txt 192.168.1.34
```

- **Confirm binary mode** before pulling kernels, initrds, or firmware to avoid silent corruption.
- **Uploads require server-side write** — ensure the TFTP root has write permission and SELinux isn't blocking the transfer before attempting a PUT.

## Security Considerations

TFTP has **no authentication and no encryption** — every request is anonymous and every byte is cleartext. From an offensive standpoint an open TFTP server is a prize: it may leak router configs, backups, or firmware, and a writable one enables uploading a malicious boot file. As a defender, treat client-side testing as an audit — if you can `get` a sensitive file without credentials, so can an attacker on the same subnet.

## Notes For Uploading Files

- Make sure the TFTP root directory (`/var/lib/tftpboot/`) has write permissions.

- SELinux might block file uploads unless policies are configured. Disable SELinux or set proper contexts for testing:

Temporarily disables SELinux

```bash
setenforce 0
```

Or use `chcon` to allow writes to the directory:

```bash
chcon -t public_content_rw_t /var/lib/tftpboot/
```

> [!TIP]
> **Prefer `chcon`/`restorecon` over disabling SELinux**
> `setenforce 0` disables SELinux system-wide and is fine for a quick lab check, but on any real host keep SELinux enforcing and grant the write context with `chcon` (or a persistent `semanage fcontext` rule) instead.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| GET hangs / times out | Firewall dropping UDP/69, or server not listening | Open `69/udp`; verify server with `ss -uln \| grep 69` |
| Downloaded file corrupted | Transferred in `netascii` mode | Set `binary` mode and retransfer |
| PUT fails with access violation | Server read-only or dir not writable | Enable `-c` on server; fix perms/SELinux context |
| File not found | File absent from TFTP root | Confirm the file exists under `/var/lib/tftpboot/` |

- Confirm the file exists in the TFTP root directory on the server.
- Ensure the firewall allows UDP port 69.
- Use `tcpdump` to debug traffic:

```bash
tcpdump -i any port 69 -n
```

- Check TFTP logs (if available) for access errors.

## References

- `man tftp` — classic TFTP client manual
- `man atftp` — advanced TFTP client manual
- RFC 1350 — The TFTP Protocol (Revision 2)

## Related

- [Trivial-File-Transfer-Protocol(TFTP)-Server](Trivial-File-Transfer-Protocol(TFTP)-Server.md) — the server this client transfers with
- [Preboot-eXecution-Environment(PXE)-Boot-Server](Preboot-eXecution-Environment(PXE)-Boot-Server.md) — PXE clients use TFTP transfers to fetch boot files
- FTP-Enumeration — closest file-transfer enumeration hub
- [Linux Administration & Server Hardening](../Readme.md) — course hub
