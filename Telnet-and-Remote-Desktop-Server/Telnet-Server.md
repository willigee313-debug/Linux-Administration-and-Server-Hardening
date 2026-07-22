# Telnet Server

## Overview

Telnet is a legacy remote-terminal protocol (TCP port **23**) that provides an interactive command-line session to a remote host. It is simple and historically ubiquitous, but it transmits **everything — including credentials — in clear text**, so it has been superseded by SSH for any real-world use.

This note documents installing the Telnet server on an RPM-based system, activating it through native **systemd socket activation** (no `xinetd`), opening the firewall, and verifying the listener. It is intended for lab, teaching, and legacy-interoperability scenarios only.

> [!WARNING]
> Telnet sends usernames, passwords, and session data unencrypted over the network. Prefer [SSH](../SSH-Secure-Shell-Server/SSH(Secure-Shell)-Server.md) for any remote access. Deploy Telnet only inside isolated, trusted lab networks.

## Concepts

| Item | Value / Meaning |
|:--|:--|
| Protocol | Telnet (clear-text terminal) |
| Port | 23/TCP |
| Server package | `telnet-server` (provides `/usr/sbin/in.telnetd`) |
| Client package | `telnet` |
| Activation model | systemd socket activation (`telnet.socket` → `telnet@.service`) |

## Install Telnet Packages

- First, check if Telnet is already installed:

```bash
rpm -qa | grep telnet
```

- If Telnet is not installed, install it along with required components:

```bash
yum install telnet*
```

- Install the specific Telnet server and client packages:

```bash
yum install telnet-server telnet
```

- Verify the installation:

```bash
rpm -qa | grep telnet
```

## Inspect Installed Packages

RPM query flags let you audit exactly what a package provides before enabling it.

| Command | Purpose |
|:--|:--|
| `rpm -qi telnet-server` | Show detailed package information |
| `rpm -ql telnet-server` | List all files provided by the package |
| `rpm -qd telnet-server` | List documentation files |
| `rpm -qc telnet-server` | List configuration files |

- Get detailed information about the Telnet server:

```bash
rpm -qi telnet-server
```

- List all files provided by the Telnet server package:

```bash
rpm -ql telnet-server
```

- List all documentation files related to the Telnet server:

```bash
rpm -qd telnet-server
```

- List configuration files associated with the Telnet server:

```bash
rpm -qc telnet-server
```

## Architecture: systemd Socket Activation

Modern systemd starts Telnet on demand: `telnet.socket` listens on port 23 and, on each incoming connection, spawns an instance of the templated `telnet@.service`.

```mermaid
flowchart LR
    C["Telnet Client"] -->|"TCP 23"| SOCK["telnet.socket"]
    SOCK -->|"Accept=yes · per-connection"| SVC["telnet@.service instance"]
    SVC --> TD["/usr/sbin/in.telnetd"]
    TD --> SH["Login shell"]
```

## Configure systemd (Native Method)

This method manages Telnet directly through systemd without relying on `xinetd`.

- Create the Telnet socket unit file:

```bash
vim /etc/systemd/system/telnet.socket
```

> Insert the following configuration:

```systemd
[Unit]
Description=Telnet Server Activation Socket

[Socket]
ListenStream=23
Accept=yes

[Install]
WantedBy=sockets.target
```

- Create the Telnet service file:

```bash
vim /etc/systemd/system/telnet@.service
```

> Insert the following content:

```systemd
[Unit]
Description=Telnet Server Service

[Service]
ExecStart=-/usr/sbin/in.telnetd
StandardInput=socket
```

### Enable and Start the Telnet Socket

- Enable the socket at boot time:

```bash
systemctl enable telnet.socket
```

- Start the socket immediately:

```bash
systemctl start telnet.socket
```

## Configure Firewall

To allow Telnet (port 23/TCP) through `firewalld`, follow these steps.

### Add Port 23/TCP Permanently

- This command adds TCP port 23 to the permanent firewall rules:

```bash
firewall-cmd --permanent --add-port=23/tcp
```

> Expected output:

```text
success
```

### Reload Firewall Rules

- Reload the firewall to apply the new settings:

```bash
firewall-cmd --reload
```

> Expected output:

```text
success
```

### Verify Open Ports

- List all open ports to confirm that port 23/TCP is active:

```bash
firewall-cmd --list-ports
```

> Example output:

```text
23/tcp
```

> If you see `23/tcp` in the output, the firewall rule has been successfully added.

## Verify Service Status

- Check if Telnet is listening on port 23:

```bash
netstat -nltup | grep 23
```

- Test the connection from a client machine:

```bash
telnet 192.168.1.33
```

## Enable Root Login via Telnet

By default, root login may be disabled over Telnet. To allow it, update the `securetty` file.

> Edit the securetty file:

```bash
vim /etc/securetty
```

> Add the following lines:

```text
pts/0
pts/1
pts/2
pts/3
pts/4
pts/5
pts/6
pts/7
pts/8
pts/9
```

> [!WARNING]
> Allowing root login over Telnet is highly discouraged. Because the session is unencrypted, a network sniffer would capture the root password in clear text. Do this only for controlled testing in an isolated environment.

## Security Considerations

- Telnet transmits **all** data, including passwords, in plain text.
- Use **SSH** instead of Telnet for secure remote access.
- If Telnet must be used, restrict access to internal, trusted networks only.
- Implement additional controls such as TCP Wrappers or IP-address restrictions.
- Keep the server disabled (`systemctl disable --now telnet.socket`) whenever it is not actively needed.

## Related
- Telnet-Enumeration — attacker-side Telnet recon
- [SSH(Secure-Shell)-Server](../SSH-Secure-Shell-Server/SSH(Secure-Shell)-Server.md) — secure replacement for Telnet
- [Remote-Desktop-Setup](Remote-Desktop-Setup.md) — remote-desktop setup overview
- [Linux Administration & Server Hardening](../Readme.md) — course hub
