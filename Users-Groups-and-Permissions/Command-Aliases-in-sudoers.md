# Command Aliases in sudoers

## Overview

Command Aliases group multiple commands under a single name in `/etc/sudoers`. This simplifies sudoers configuration by letting you specify sets of commands once, then grant users or groups permission to run those sets without listing each command individually. Aliases keep large policies readable and reduce the risk of inconsistent, copy-pasted rules.

## Concepts

`sudoers` supports four alias types, all defined with an uppercase keyword:

| Alias Type | Keyword | Groups together |
| :-- | :-- | :-- |
| User Alias | `User_Alias` | Users, groups (`%group`), UIDs |
| Command Alias | `Cmnd_Alias` | Absolute command paths |
| Host Alias | `Host_Alias` | Hostnames, IPs, networks |
| Runas Alias | `Runas_Alias` | Target users/groups a command may run as |

This note focuses on **`Cmnd_Alias`** — a named set of commands referenced by the privilege specification lines.

> [!IMPORTANT]
> Alias names must be in **UPPERCASE**. Command paths inside a `Cmnd_Alias` should be **absolute** (e.g., `/usr/bin/yum`), never bare command names, so a user cannot substitute a malicious binary earlier in `PATH`.

## Configuration

### Editing sudoers Safely

Always edit the sudoers file with `visudo`, which performs a syntax check before saving and prevents concurrent edits:

```bash
visudo
```

Or with the vim editor:

```bash
EDITOR=vim visudo
```

> [!WARNING]
> Never edit `/etc/sudoers` directly with a plain editor. A syntax error can lock every user out of `sudo`. `visudo` validates the file and refuses to install a broken policy.

### Example sudoers with Command Aliases and User Aliases

```bash
# User Aliases
User_Alias IT_USERS = it1, it2, it3
User_Alias HR_USERS = hr1, hr2, hr3

# Command Aliases
Cmnd_Alias SOFTWARE      = /bin/rpm, /usr/bin/up2date, /usr/bin/yum
Cmnd_Alias ARMOUR_CMD    = /usr/sbin/fdisk, /usr/bin/yum, /usr/sbin/useradd
Cmnd_Alias INFOSEC_CMD   = /usr/bin/id, /usr/bin/whoami, /usr/bin/mkdir
Cmnd_Alias IT_CMDS       = /usr/sbin/ifconfig, /usr/bin/ping, /usr/sbin/ip, /usr/bin/vim, /usr/bin/rpm

# Defaults Section
Defaults        !visiblepw
Defaults        always_set_home
Defaults        match_group_by_gid
Defaults        always_query_group_plugin
Defaults        env_reset
Defaults        env_keep = "COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS"
Defaults        env_keep += "MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
Defaults        env_keep += "LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES"
Defaults        env_keep += "LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE"
Defaults        env_keep += "LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"
Defaults        secure_path = /sbin:/bin:/usr/sbin:/usr/bin

# Privilege Specifications
root        ALL=(ALL) ALL
armour      ALL=(ALL) /usr/bin/id
IT_USERS    ALL=(root, armour, "hr1":"HR") SOFTWARE, ARMOUR_CMD, INFOSEC_CMD
HR_USERS    ALL=(root, armour, "it1":"IT") SOFTWARE, IT_CMDS
%wheel      ALL=(ALL) ALL
```

### Anatomy of a Privilege Specification Line

The rule `IT_USERS ALL=(root, armour, "hr1":"HR") SOFTWARE, ARMOUR_CMD, INFOSEC_CMD` reads as:

| Element | Value | Meaning |
| :-- | :-- | :-- |
| Who | `IT_USERS` | The user alias the rule applies to |
| Host | `ALL` | On which hosts the rule is valid |
| Runas users/groups | `(root, armour, "hr1":"HR")` | Identities the command may run as |
| Commands | `SOFTWARE, ARMOUR_CMD, INFOSEC_CMD` | The command aliases permitted |

## Explanation

- `User_Alias` defines user groups like `IT_USERS` and `HR_USERS`.
- `Cmnd_Alias` defines groups of commands such as `SOFTWARE`, `ARMOUR_CMD`, etc.
- Permissions are assigned by associating user aliases with command aliases.
- **Example**: Members of `IT_USERS` can run commands from `SOFTWARE`, `ARMOUR_CMD`, and `INFOSEC_CMD` as either `root`, `armour`, or `hr1` with group `HR`.
- `armour` can only run `/usr/bin/id` anywhere.
- Members of `HR_USERS` can run several command groups as `root`, `armour`, or `it1` (group `IT`).

## Examples

List allowed sudo commands for the current user:

```bash
sudo -l
```

Run the `id` command as user armour:

```bash
sudo -u armour id
```

Run the `id` command as root:

```bash
sudo -u root id
```

Run fdisk (part of `ARMOUR_CMD`) to list disks — requires appropriate sudo privileges:

```bash
sudo /usr/sbin/fdisk -l
```

Add a new user named dd1 (requires `useradd` permission):

```bash
sudo /usr/sbin/useradd dd1
```

## Security Considerations

- **Beware of privesc-enabling commands.** Aliases that include editors (`/usr/bin/vim`), package managers (`/usr/bin/yum`, `/bin/rpm`), or account tools (`/usr/sbin/useradd`) effectively grant root. `vim` can spawn a shell (`:!sh`); `yum`/`rpm` can run scriptlets; `useradd` can create a UID 0 account. Treat these as full root grants.
- **Always use absolute paths** in `Cmnd_Alias` entries and set `secure_path` so a user cannot hijack a relative command name.
- **Prefer specific over broad.** `ALL` in a command field is equivalent to unrestricted root; scope aliases to the minimum commands a role needs (least privilege, per CIS Benchmark guidance).
- **Audit runas targets.** The runas list `(root, armour, ...)` widens who a user can impersonate; keep it as narrow as the task requires.
- **Review regularly.** Run `sudo -l` per account and audit `Cmnd_Alias` definitions during access reviews to catch drift.

## Troubleshooting

| Symptom | Likely cause | Fix |
| :-- | :-- | :-- |
| `visudo` reports a parse error | Lowercase alias name or missing absolute path | Use UPPERCASE alias names and absolute command paths |
| User cannot run an aliased command | Command not listed in the alias, or wrong host field | Verify with `sudo -l`; confirm the `Cmnd_Alias` and host match |
| `command not found` under sudo | `secure_path` excludes the binary's directory | Add the directory to `Defaults secure_path` |
| Changes not taking effect | Edited a copy or an included file, not the active policy | Confirm the file and any `@includedir /etc/sudoers.d` fragments |

## Related

- [Sudo](Sudo.md) — parent sudo command
- [User-Aliases-in-sudoers](User-Aliases-in-sudoers.md) — related sudoers alias type
- [Host-Aliases-in-sudoers](Host-Aliases-in-sudoers.md) — related sudoers alias type
- [Understanding-NOPASSWD-in-sudoers](Understanding-NOPASSWD-in-sudoers.md) — passwordless sudo directive
- [Real-World-Examples-Combine-Aliases](Real-World-Examples-Combine-Aliases.md) — aliases combined in real rules
- Privilege-Escalation — misconfigured Cmnd_Alias is a privesc vector
- [Linux Administration & Server Hardening](../Readme.md) — course hub.
