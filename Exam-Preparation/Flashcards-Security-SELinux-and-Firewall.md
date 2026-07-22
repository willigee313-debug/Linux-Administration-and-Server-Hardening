# Flashcards — Security, SELinux & Firewall

Spaced-repetition cards covering SELinux (modes, contexts, booleans/ports, troubleshooting), host firewalls (firewalld, iptables, nftables, ufw), tcpdump, ClamAV, and GRUB/root-password recovery — pulled from the Security, Firewall and Monitoring module for RHCSA/LFCS/Linux+/LPIC-1 prep.

## SELinux Fundamentals

Which command shows the current SELinux enforcement mode (Enforcing/Permissive/Disabled)?::`getenforce`
Which two values does `setenforce` accept, and what can it never set?::`0` (Permissive) and `1` (Enforcing) — it cannot set `Disabled`; that requires editing the config file and rebooting
Which file sets the persistent SELinux mode and policy type applied at next boot?::`/etc/selinux/config` (`SELINUX=` and `SELINUXTYPE=`)
What flag file, created before a reboot, forces a full filesystem SELinux relabel?::`/.autorelabel`

## SELinux Contexts and File Labeling

What are the four colon-separated fields of an SELinux security context?::`user:role:type:level`
Which command displays a file's SELinux security context?::`ls -Z`
Which relabeling command is temporary (reverts on next relabel) vs. which two commands make a persistent file-context change?::`chcon` is temporary; `semanage fcontext -a -t <type> "<regex>"` + `restorecon -Rv <path>` is persistent
When moving a file into a web root with `mv` (same filesystem), does it get the destination directory's default SELinux type or keep its original label?::It keeps the source's original label — `mv` on the same filesystem only renames the inode, it doesn't relabel (unlike `cp`, which gets the destination's default type)

## SELinux Booleans and Ports

Which command lists every SELinux boolean and its current state?::`getsebool -a`
What flag on `setsebool` makes a boolean change persist across reboot?::`-P` (e.g. `setsebool -P httpd_can_network_connect on`)
Which command adds TCP port 8080 to the `http_port_t` SELinux port type so httpd can bind it?::`semanage port -a -t http_port_t -p tcp 8080`

## SELinux Troubleshooting

Where does the kernel log AVC (Access Vector Cache) denials?::`/var/log/audit/audit.log`
Which command filters the audit log for recent AVC/USER_AVC denials?::`ausearch -m AVC,USER_AVC -ts recent`
Which command puts a single SELinux domain into permissive mode for diagnosis without disabling SELinux globally?::`semanage permissive -a <domain>` (e.g. `httpd_t`)

## Firewalld

Which command applies permanent firewalld rule changes without restarting the service?::`firewall-cmd --reload`
Which flag must accompany a `firewall-cmd` rule so it survives a reload/reboot instead of only applying at runtime?::`--permanent`

## IPTables

Which three built-in chains make up iptables' `filter` table?::`INPUT`, `OUTPUT`, `FORWARD`
What is the behavioral difference between the `DROP` and `REJECT` targets?::`DROP` silently discards the packet (sender sees a timeout); `REJECT` returns an ICMP error immediately
Which command persists the running iptables ruleset on RHEL-family systems so it survives a reboot?::`service iptables save` (or `iptables-save > /etc/sysconfig/iptables`)

## nftables

What single command-line utility does nftables use in place of iptables' separate per-family binaries (`iptables`, `ip6tables`, `arptables`, `ebtables`)?::`nft`
Which nftables address family covers IPv4 and IPv6 together in one ruleset, and is the recommended default for host firewalls?::`inet`
Which command validates an nftables ruleset file's syntax without actually loading it?::`nft -c -f <file>`

## Uncomplicated Firewall (ufw)

Which command must be run before `ufw enable` on a remote host to avoid an SSH lockout?::`ufw allow ssh`
What is the difference between ufw's `deny` and `reject` rule actions?::`deny` silently drops traffic (port appears filtered); `reject` returns an ICMP/RST (port appears closed)

## TCPDump

Which tcpdump option writes captured packets to a pcap file for later analysis?::`-w <file>`
Which tcpdump option suppresses both hostname and port-number resolution?::`-nn`

## ClamAV Antivirus

Which command updates ClamAV's virus signature database?::`freshclam`
What is the EICAR test file used for?::A harmless, standardized string that every AV engine (including ClamAV) flags as malware, used to validate detection without using real malware

## GRUB & Root Password Recovery

Which kernel parameter, appended in the GRUB edit screen, drops a RHEL/CentOS/Fedora system into an emergency shell for root password reset (SELinux-aware method)?::`rd.break`
Which command is the recommended way to set a GRUB boot loader password interactively (storing the hash in `/boot/grub2/user.cfg`)?::`grub2-setpassword`

## Related

- [Security, Firewall and Monitoring](../Security-Firewall-and-Monitoring/Readme.md)
- [Exam Preparation](Readme.md)
- [Linux Administration & Server Hardening](../Readme.md)
