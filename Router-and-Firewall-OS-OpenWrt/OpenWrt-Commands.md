# OpenWrt Commands

## Overview

Comprehensive command reference for managing OpenWrt via terminal or SSH. It covers system monitoring, network and wireless configuration, UCI, the firewall, package and service management, disk utilities, logging, backup/restore, diagnostics, and access control. Most configuration is driven through **UCI** (`uci get/set/commit`) against files in `/etc/config/`, with an `/etc/init.d/<service>` reload applying the change.

> [!TIP]
> **The UCI change cycle**
> Almost every config change follows the same three steps: `uci set …` to stage, `uci commit <config>` to persist, then a service reload (`/etc/init.d/<service> reload|restart`) to apply. An uncommitted change is lost on reboot.

## System Information

Shows how long the system has been running, along with load averages.

```bash
uptime
```

Interactive process viewer to monitor system usage in real-time.

```bash
top
```

Displays memory usage in a human-readable format.

```bash
free -h
```

## Network Configuration

Lists all network interfaces and their IP/MAC addresses.

```bash
ip a
```

Displays current routing table.

```bash
ip r
```

Returns JSON output showing the LAN interface status.

```bash
ifstatus lan
```

Prints current UCI-managed network configuration.

```bash
uci show network
```

Restarts all network services and applies changes.

```bash
/etc/init.d/network restart
```

Changes LAN IP address and applies the new configuration.

```bash
uci set network.lan.ipaddr='192.168.1.100'
```

```bash
uci commit network
```

```bash
/etc/init.d/network restart
```

## Wireless Configuration

Displays current wireless state in JSON format.

```bash
wifi status
```

Reloads wireless configuration without restarting the whole network.

```bash
wifi reload
```

Shows wireless-related configurations such as SSID, encryption, etc.

```bash
uci show wireless
```

Displays wireless interface info including signal strength and bitrate.

```bash
iwinfo
```

## UCI Configuration

Retrieves the value of a specific configuration option.

```bash
uci get <config>.<section>.<option>
```

Modifies a configuration option.

```bash
uci set <config>.<section>.<option>=<value>
```

Commits the change to make it persistent.

```bash
uci commit <config>
```

Example: Enable wireless interface and apply configuration.

```bash
uci set wireless.@wifi-iface[0].disabled=0
```

```bash
uci commit wireless
```

```bash
wifi reload
```

## Firewall Configuration

Displays firewall configuration managed via UCI.

```bash
uci show firewall
```

Restarts the firewall to apply any changes.

```bash
/etc/init.d/firewall restart
```

Lists all iptables rules (non-UCI level), useful for diagnostics.

```bash
iptables -L -v -n
```

Displays rules if nftables is in use.

```bash
nft list ruleset
```

## Package Management

Refreshes the package list from OpenWrt repositories.

```bash
opkg update
```

Installs a new package.

```bash
opkg install <package>
```

Uninstalls a package and its files.

```bash
opkg remove <package>
```

Lists all currently installed packages.

```bash
opkg list-installed
```

Upgrade Each Package

```bash
opkg list-installed | cut -f1 -d' ' | xargs -r opkg upgrade
```

Displays files installed by a specific package.

```bash
opkg files <package>
```

Prints detailed info about a package.

```bash
opkg info <package>
```

## Service Management

Starts the specified service.

```bash
/etc/init.d/<service> start
```

Stops the specified service.

```bash
/etc/init.d/<service> stop
```

Restarts the specified service.

```bash
/etc/init.d/<service> restart
```

Enables the service to start on boot.

```bash
/etc/init.d/<service> enable
```

Prevents the service from starting on boot.

```bash
/etc/init.d/<service> disable
```

## File and Disk Utilities

Shows disk usage of mounted filesystems in human-readable format.

```bash
df -h
```

Lists all UCI-managed configuration files.

```bash
ls /etc/config/
```

Manually edit a configuration file.

```bash
vi /etc/config/network
```

Checks space used in overlay filesystem.

```bash
du -sh /overlay
```

## Logs and Debugging

Displays the system log buffer.

```bash
logread
```

Shows kernel-related messages from boot or runtime.

```bash
dmesg
```

Follows the system log in real-time.

```bash
logread -f
```

Reads persistent logs if enabled.

```bash
cat /var/log/messages
```

## Reboot and Backup

Reboots the router.

```bash
reboot
```

Resets configuration to factory defaults.

```bash
firstboot
```

Alternative factory reset that erases overlay data.

```bash
jffs2reset && reboot
```

Flashes a new firmware image.

```bash
sysupgrade /tmp/firmware.bin
```

Creates a manual backup of all config files.

```bash
tar -cvzf /tmp/backup.tar.gz /etc/config
```

Restores backup.

```bash
tar -xvzf /tmp/backup.tar.gz -C /
```

## Diagnostics and Testing

Sends ICMP echo requests to test connectivity.

```bash
ping <host>
```

Displays the path packets take to reach the host.

```bash
traceroute <host>
```

Resolves a domain name to an IP address.

```bash
nslookup <host>
```

Tests HTTP connectivity and content fetch.

```bash
wget -O - http://example.com
```

Captures packets for network analysis (if installed).

```bash
tcpdump -i any
```

## User and Access Control

Changes the root password.

```bash
passwd
```

Lists user accounts.

```bash
cat /etc/passwd
```

Lists user encrypted passwords (restricted view).

```bash
cat /etc/shadow
```

Generates a new RSA SSH key for Dropbear.

```bash
dropbearkey -t rsa -f /etc/dropbear/dropbear_rsa_host_key
```

## Related
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub.
- [Router and Firewall OS (OpenWrt)](Readme.md) — module hub.
- [Firewall-Rules-in-OpenWrt](Firewall-Rules-in-OpenWrt.md) — applies these commands to firewall configuration.
- [OpenWrt-Upgrade](OpenWrt-Upgrade.md) — upgrade procedure driven from the CLI.
