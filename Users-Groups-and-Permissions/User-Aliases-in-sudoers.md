# User Aliases in sudoers

## Overview

**User Aliases** allow grouping multiple users under a single name in the `/etc/sudoers` file. This simplifies privilege management by assigning permissions to a group alias instead of repeating rules for individual users. When the membership of a role changes, you edit one `User_Alias` definition instead of hunting down every rule that mentions the affected accounts — reducing both effort and the chance of an inconsistent, exploitable policy.

> [!NOTE]
> A `User_Alias` is a **sudoers construct**, not a Linux group. It exists only inside the sudoers policy. Linux groups (referenced with a leading `%`, e.g. `%wheel`) are managed with `groupadd`/`gpasswd` and are a separate mechanism you can also use in rules.

## Concepts

| Alias type | Keyword | Groups |
| --- | --- | --- |
| User alias | `User_Alias` | Users who receive privileges |
| Runas alias | `Runas_Alias` | Identities commands may be run *as* |
| Host alias | `Host_Alias` | Hosts a rule applies to |
| Command alias | `Cmnd_Alias` | Commands a rule permits |

This note focuses on `User_Alias`. Alias names must be **uppercase**, start with a letter, and contain only letters, digits, and underscores.

## Configuration

### Editing the sudoers File

Always use `visudo` to edit the sudoers file safely and avoid syntax errors:

```bash
visudo
```

- Or with vim as the editor:

```bash
EDITOR=vim visudo
```

> [!WARNING]
> Never edit `/etc/sudoers` directly with a plain editor. `visudo` validates the grammar (including alias definitions) before saving; an invalid alias can break every rule that references it.

### Example User Alias Definitions

```bash
User_Alias IT_USERS = it1, it2, it3
```

```bash
User_Alias ARMOUR_USER = armour, it1, u1
```

- `IT_USERS` groups three users: `it1`, `it2`, and `it3`.
- `ARMOUR_USER` groups `armour`, `it1`, and `u1`.

### Typical sudoers Configuration Using User Aliases

```bash
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
root            ALL=(ALL) ALL
%wheel          ALL=(ALL) ALL
IT_USERS        ALL=(ALL) ALL
ARMOUR_USER     ALL=(armour, hr1, emp1) ALL
u1              ALL=(armour, infosec, it1:root) ALL
```

- `IT_USERS` can run any command as any user on all hosts
- `ARMOUR_USER` alias can run commands as specific other users
- User `u1` may run commands as those users and root group

> [!WARNING]
> `IT_USERS ALL=(ALL) ALL` grants the whole alias unrestricted root on every host — the sudoers equivalent of handing out the root password. Aliases make broad grants concise, which also makes over-privilege easy to introduce accidentally. Scope the `COMMANDS` field to a `Cmnd_Alias` wherever possible.

## Examples

### Complex User Alias Usage

- `IT_USERS` can run commands as user `armour`, or as user `hr1` with group `HR`.

```bash
IT_USERS ALL=(armour, "hr1":"HR") ALL
```

- `IT_USERS` can run commands with group privileges `HR`, `IT`, or `EMP`.

```bash
IT_USERS ALL=(:HR, IT, EMP) ALL
```

### Useful sudo Commands Using Users and Groups

- List allowed sudo commands for the current user

```bash
sudo -l
```

- Show current user id

```bash
id
```

- Run id command as user it2

```bash
sudo -u it2 id
```

- Run id command with group IT

```bash
sudo -g IT id
```

- Run mkdir as user it2 with group demo3

```bash
sudo -u it2 -g demo3 mkdir /tmp/a3
```

```bash
sudo -u it2 -g demo3 mkdir /home/it2/a1
```

## Best Practices

> [!TIP]
> - Name aliases after **roles** (`DBA_USERS`, `BACKUP_OPERATORS`), not individuals, so rules stay stable as staff change.
> - Pair `User_Alias` with a `Cmnd_Alias` to keep grants least-privilege: `DBA_USERS ALL = DB_CMDS` rather than `... ALL`.
> - Keep a single source of truth: define aliases once, near the top of the file, and reference them below.
> - Review alias membership during access recertification — a stale name left in an alias is silent standing privilege.

## Security Considerations

Aliases are convenient but they also **hide the effective blast radius** of a rule: a single line like `IT_USERS ALL=(ALL) ALL` may grant root to many accounts whose membership is defined elsewhere in the file. During a privilege-escalation review, always expand every alias to its concrete users and commands before judging a rule safe. From a target host, `sudo -l` shows the *resolved* privileges for the current user, which is what an attacker will rely on regardless of how the policy was written.

## Summary

- **User_Alias** defines named groups of users.
- Aliases simplify sudoers configuration for multiple users.
- You can combine user and group specifications for granular control.
- Use `sudo -u USER -g GROUP` to run commands as specific users and groups.

## Related

- [Sudo](Sudo.md) — parent sudo command and policy model.
- [Command-Aliases-in-sudoers](Command-Aliases-in-sudoers.md) — group commands to keep alias grants least-privilege.
- [Host-Aliases-in-sudoers](Host-Aliases-in-sudoers.md) — restrict rules to specific hosts.
- [Real-World-Examples-Combine-Aliases](Real-World-Examples-Combine-Aliases.md) — aliases combined in complete rules.
- [Understanding-NOPASSWD-in-sudoers](Understanding-NOPASSWD-in-sudoers.md) — passwordless variants of these rules.
- Privilege-Escalation — sudoers misconfiguration as a privesc vector.
- [Linux Administration & Server Hardening](../Readme.md) — course hub.
