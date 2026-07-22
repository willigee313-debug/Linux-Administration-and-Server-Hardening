# Flashcards — Networking

Deck covering Linux network interface configuration, diagnostics, monitoring, routing, time sync, VLANs/bonding, and SSH server hardening for RHCSA/LFCS/Linux+/LPIC-1 prep.

## Interfaces & Addressing

Which command shows all network interfaces including inactive ones with legacy tooling?::`ifconfig -a`
What is the modern `ip` command to display all interfaces and their addresses?::`ip addr show` (or `ip a`)
What command adds a temporary (non-persistent) IPv4 address to an interface with `ip`?::`ip addr add 192.168.1.40/24 dev enp0s3`
Why are `ip addr add` / `ip route add` changes lost after a reboot?::They only modify the live kernel state; persistence requires editing the distro's backend config (NetworkManager, ifcfg, or `/etc/network/interfaces`)
Which file holds NetworkManager's persistent connection profiles?::`/etc/NetworkManager/system-connections/*.nmconnection`
What permissions must a hand-edited `.nmconnection` keyfile have, and why does it matter?::`600` owned by `root` — NetworkManager refuses to load world/group-readable profiles since they can contain secrets
On CentOS 7/RHEL 7, where are legacy per-interface network scripts stored?::`/etc/sysconfig/network-scripts/ifcfg-*`
Which file configures interfaces on Debian/Ubuntu's traditional ifupdown system?::`/etc/network/interfaces`
What YAML-based configuration system does Ubuntu Server use, and what command applies it?::netplan, applied with `netplan apply`

## nmcli

What `nmcli` command creates a new static Ethernet connection profile named `static-eth0` on `enp0s3`?::`nmcli con add type ethernet con-name static-eth0 ifname enp0s3 ipv4.addresses 192.168.1.10/24 ipv4.gateway 192.168.1.1 ipv4.dns 1.1.1.1 ipv4.method manual`
What does `nmcli con mod <name> +ipv4.addresses <ip>` do differently from `nmcli con mod <name> ipv4.addresses <ip>` (no `+`)?::The `+` prefix appends to a multi-value property instead of replacing it
Which `nmcli` command applies profile changes to a running interface without a full disconnect/reconnect?::`nmcli device reapply <interface>`
What `nmcli con mod` property must be set to `manual` for a static address to actually take effect?::`ipv4.method` (leaving it `auto` means DHCP still applies)

## Diagnostics

What protocol does `ping` use to test reachability?::ICMP Echo Request/Reply
Why can a failed `ping` NOT prove a host is down?::Many hardened hosts/firewalls drop ICMP by policy while still serving TCP applications
Which tool traces the network path without requiring root privileges and also discovers path MTU?::`tracepath`
Which tool traces a path using TCP SYN segments to get through firewalls that block ICMP/UDP?::`tcptraceroute`
What `ping` option stops the command after sending a fixed number of packets?::`-c <n>`

## Monitoring Sockets

What is the single most useful `ss` command combination for auditing a server's exposed TCP surface?::`ss -ltnp` (listening TCP, numeric, with owning process)
Why is `ss` generally preferred over `netstat` on modern systems?::It reads socket data directly from the kernel via Netlink (faster) and is pre-installed, while `netstat` requires the deprecated `net-tools` package

## Routing & Forwarding

Which sysctl parameter must be set to `1` to make a Linux host route (forward) packets between interfaces?::`net.ipv4.ip_forward`
What command performs a dry-run route lookup without sending any packets?::`ip route get <destination>`
Why is `ip route replace` preferred over `ip route add` in automation scripts?::It is idempotent — it won't fail with "File exists" if the route already exists

## Time Sync (chrony)

Which daemon and control utility make up the chrony NTP service?::`chronyd` (daemon) and `chronyc` (control tool)
What does the `chrony.conf` directive `makestep 1.0 3` do?::Steps (jumps) the clock instead of slowly slewing it if the offset exceeds 1.0s during the first 3 clock updates
What `chronyc` subcommand shows synchronization status including offset and Leap status?::`chronyc tracking`

## VLANs & Bonding

Which bonding mode requires no special switch configuration and is the safest default?::Mode 1, `active-backup`
Which bonding mode requires the switch to be configured for LACP/802.3ad port-channel?::Mode 4, `802.3ad` (LACP)

## SSH Server Hardening

Which `sshd_config` directive restricts which interface/IP address `sshd` binds to?::`ListenAddress`
Which `sshd_config` directive changes the port `sshd` listens on?::`Port`
What is the recommended production value for `PermitRootLogin` per CIS/NIST guidance?::`no`
What `sshd_config` value allows root to log in with keys only, never a password?::`PermitRootLogin prohibit-password`
Which `sshd_config` directives restrict SSH login by user (optionally bound to a source IP)?::`AllowUsers` and `DenyUsers`
In OpenSSH access-control evaluation, which directive type always wins over `Allow` rules?::`Deny` rules (evaluated in order `DenyUsers` → `AllowUsers` → `DenyGroups` → `AllowGroups`)
What command validates `sshd_config` syntax before restarting the daemon (to avoid lockout)?::`sshd -t`
On a SELinux-enforcing RHEL system, what command must be run after changing the SSH port so `sshd` can bind to it?::`semanage port -a -t ssh_port_t -p tcp <port>`
Why can TCP Wrappers (`/etc/hosts.allow`/`/etc/hosts.deny`) commonly have no effect on modern `sshd`?::Support for libwrap/TCP Wrappers was removed from OpenSSH as of version 6.7; current `sshd` is typically not libwrap-linked
Which key type is recommended over legacy RSA/DSA for new SSH key pairs?::Ed25519 (`ssh-keygen -t ed25519`)
What ownership/permission issue is the most common cause of key-based auth silently failing?::`~/.ssh` and/or `authorized_keys` not owned by the target user or writable by others (StrictModes rejects it)

## Related
- [Network Configuration](../Network-Configuration/Readme.md)
- [SSH Secure Shell Server](../SSH-Secure-Shell-Server/Readme.md)
- [Exam Preparation](Readme.md)
- [Linux Administration & Server Hardening](../Readme.md)
