# Host Aliases in sudoers

## Overview

Host Aliases in `/etc/sudoers` let you group several hostnames, IP addresses, or networks under a single uppercase name. When the same `/etc/sudoers` policy is distributed to many machines (for example, from a central configuration-management system), a Host Alias lets one rule apply on some hosts but not others. This keeps a shared policy DRY and makes "who can do what, and *where*" explicit and auditable.

> [!NOTE]
> The host field only matters when the *same* sudoers file is deployed across multiple systems. On a standalone host, `ALL` in the host field is effectively the only value that ever matches, and Host Aliases add no benefit.

## Concepts

`sudoers` supports four alias types, each declared with an uppercase keyword. Host Aliases are one of them:

| Alias type | Keyword | Groups together |
| :-- | :-- | :-- |
| User Alias | `User_Alias` | Users, groups (`%group`), UIDs |
| Runas Alias | `Runas_Alias` | Target users/groups a command may run as |
| Host Alias | `Host_Alias` | Hostnames, IPs, networks, netgroups |
| Command Alias | `Cmnd_Alias` | Absolute command paths |

A `Host_Alias` member can be:

- A hostname (`webserver`, `ns1`) — matched against the machine's own hostname.
- An IP address (`10.0.0.5`).
- A network in CIDR or netmask form (`10.0.0.0/24`, `192.168.1.0/255.255.255.0`).
- An NIS netgroup (`+netgroup`).

> [!IMPORTANT]
> Alias names must be **UPPERCASE**. Matching hostnames requires that `sudo` can resolve the local host consistently — prefer IPs/CIDRs over short hostnames to avoid DNS or `/etc/hosts` ambiguity when the policy is shared across machines.

## Configuration

### Editing sudoers Safely

Always edit the sudoers file with `visudo`, which validates syntax before saving and prevents concurrent edits:

```bash
sudo visudo
```

Or specify your preferred editor, for example vim:

```bash
EDITOR=vim visudo
```

This opens `/etc/sudoers` safely and performs a syntax check before writing the file back.

> [!WARNING]
> Never edit `/etc/sudoers` with a plain editor. A single syntax error can lock every account out of `sudo`. `visudo` refuses to install a broken policy.

### Defining a Host Alias

```bash
Host_Alias IT_HOST = ns1, webserver, localhost
```

- Defines `IT_HOST` as an alias for the hosts named `ns1`, `webserver`, and `localhost`.
- You can then use `IT_HOST` in the host field of any rule to apply it to all of these hosts at once.

## Examples

### Complete sudoers Configuration with Host Aliases and Defaults

```bash
Defaults    !visiblepw
Defaults    always_set_home
Defaults    match_group_by_gid
Defaults    always_query_group_plugin
Defaults    env_reset
Defaults    env_keep = "COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS"
Defaults    env_keep += "MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
Defaults    env_keep += "LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES"
Defaults    env_keep += "LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE"
Defaults    env_keep += "LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin
root    ALL=(ALL) ALL
armour  IT_HOST=(ALL) ALL
%wheel  ALL=(ALL) ALL
```

### Reading the Rules

| Rule | Meaning |
| :-- | :-- |
| `root ALL=(ALL) ALL` | `root` can run any command, as any user, on all hosts. |
| `armour IT_HOST=(ALL) ALL` | `armour` can run any command as any user, but only on hosts in `IT_HOST` (`ns1`, `webserver`, `localhost`). |
| `%wheel ALL=(ALL) ALL` | Members of the `wheel` group can run any command on all hosts. |

- The alias `IT_HOST` groups the hosts `ns1`, `webserver`, and `localhost`.
- The rule for `armour` applies **only** on hosts in the `IT_HOST` group; on any other host the rule does not match and `sudo` is denied.
- The `%wheel` group retains access on all hosts.
- The `Defaults` entries configure environment protection (`env_reset`, `env_keep`, `secure_path`) and other hardening features for every `sudo` session.

### Verifying What Applies on This Host

List the current user's effective sudo rights on the local machine:

```bash
sudo -l
```

## Security Considerations

- **Host aliases are not a security boundary by themselves.** A user who can log in to a host listed in the alias can use the rule there; restrict *which* hosts a role needs, following least privilege (CIS Benchmark guidance).
- **Prefer IP/CIDR over hostnames.** Hostname matching depends on name resolution; an attacker who can influence DNS or `/etc/hosts` could change which rules match. Pinning to IPs/CIDRs removes that ambiguity.
- **Avoid broad `ALL` command grants** even when scoped by host — `IT_HOST=(ALL) ALL` still grants full root on those machines. Scope the command field with a `Cmnd_Alias` where possible.
- **Audit centrally distributed policies.** When one sudoers file is pushed to a fleet, review Host Aliases during access reviews so a rule doesn't silently apply to newly provisioned hosts.

## Troubleshooting

| Symptom | Likely cause | Fix |
| :-- | :-- | :-- |
| `visudo` reports a parse error | Lowercase alias name or bad member syntax | Use an UPPERCASE alias name and valid host/IP/CIDR members |
| Rule doesn't apply on a host you expect | Hostname doesn't match the alias member | Confirm the local hostname (`hostname`); prefer IP/CIDR members |
| Rule applies where it shouldn't | Overly broad CIDR or `ALL` in host field | Narrow the network range; scope the alias to specific hosts |
| Rule ignored after edits | Edited a copy or a `sudoers.d` fragment, not the active file | Confirm the active file and any `@includedir /etc/sudoers.d` |

## Related

- [Sudo](Sudo.md) — parent sudo command
- [Command-Aliases-in-sudoers](Command-Aliases-in-sudoers.md) — group commands under one alias
- [User-Aliases-in-sudoers](User-Aliases-in-sudoers.md) — group users under one alias
- [Real-World-Examples-Combine-Aliases](Real-World-Examples-Combine-Aliases.md) — aliases combined in real rules
- [Understanding-NOPASSWD-in-sudoers](Understanding-NOPASSWD-in-sudoers.md) — passwordless sudo directive
- Privilege-Escalation — sudoers misconfiguration is a privesc vector
- [Linux Administration & Server Hardening](../Readme.md) — course hub.
