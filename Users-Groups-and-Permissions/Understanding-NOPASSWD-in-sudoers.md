# Understanding NOPASSWD in sudoers

## Overview

The `NOPASSWD` tag in a sudoers rule allows specified users or groups to execute certain commands *without* being prompted to enter their password.

This is useful for:

- Automating scripts or services that require sudo without manual intervention.
- Reducing friction for frequently used administrative commands while maintaining privilege control.

> [!WARNING]
> `NOPASSWD` removes the last authentication checkpoint before privileged execution. If an attacker gains a shell as a `NOPASSWD`-enabled user, every command covered by that tag runs as root instantly — no password, no re-authentication. Scope it to the narrowest, safest command set possible and never apply it to `ALL`.

## Concepts

| Tag | Effect |
| --- | --- |
| `PASSWD:` | Default behaviour — the invoking user must authenticate |
| `NOPASSWD:` | Skips the password prompt for the commands that follow it |

Tags are **positional and sticky**: a tag applies to every command listed after it until another tag overrides it on the same line. This is why `NOPASSWD:` and `PASSWD:` are often combined in one rule to grant passwordless access to some commands while still requiring a password for others.

## Syntax Overview

```bash
USER  HOST=(RUNAS_USER) [NOPASSWD:] COMMANDS
```

- Placing `NOPASSWD:` before commands means no password prompt is required for those commands.
- You can mix `NOPASSWD:` and password-required commands by defining permissions accordingly.

## Configuration

### Example sudoers Snippet with NOPASSWD

```bash
User_Alias IT_USERS = it1, it2, it3
User_Alias HR_USERS = hr1, hr2, hr3

Cmnd_Alias SOFTWARE    = /bin/rpm, /usr/bin/up2date, /usr/bin/yum
Cmnd_Alias ARMOUR_CMD  = /usr/sbin/fdisk, /usr/bin/yum, /usr/sbin/useradd
Cmnd_Alias INFOSEC_CMD = /usr/bin/id, /usr/bin/whoami, /usr/bin/mkdir
Cmnd_Alias IT_CMDS     = /usr/sbin/ifconfig, /usr/bin/ping, /usr/sbin/ip, /usr/bin/vim, /usr/bin/rpm

Defaults   !visiblepw
Defaults   always_set_home
Defaults   match_group_by_gid
Defaults   always_query_group_plugin
Defaults   env_reset
Defaults   env_keep =  "COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS"
Defaults   env_keep += "MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
Defaults   env_keep += "LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES"
Defaults   env_keep += "LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE"
Defaults   env_keep += "LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"
Defaults   secure_path = /sbin:/bin:/usr/sbin:/usr/bin

root ALL=(ALL) ALL
armour ALL=(ALL) /usr/bin/id

# IT_USERS can run SOFTWARE commands as root without needing password
IT_USERS ALL=(root) NOPASSWD: SOFTWARE

# HR_USERS can run SOFTWARE and ARMOUR_CMD without password, but require password for IT_CMDS
HR_USERS ALL=(ALL) NOPASSWD: SOFTWARE, ARMOUR_CMD, PASSWD: IT_CMDS

%wheel ALL=(ALL) ALL
```

### Explanation

- `IT_USERS` (it1, it2, it3) run software management commands (`rpm`, `yum`, `up2date`) as root **without password prompts**.
- `HR_USERS` (hr1, hr2, hr3) can run software commands and armour commands **without password**, but must enter their password to run `IT_CMDS` like networking tools and editors.
- Members of `%wheel` group have normal sudo access requiring passwords as per default rules.

> [!WARNING]
> Granting `/usr/bin/vim` (as in `IT_CMDS`) via sudo — even *with* a password — is dangerous: `vim` can spawn a root shell (`:!/bin/sh`). The password requirement only slows an attacker who already has the user's credentials; it does not contain the breakout. See GTFOBins for the full list of such binaries.

## Examples

### Other Common Uses of NOPASSWD

```bash
armour ALL=(ALL:ALL) ARMOUR_CMD
Rahul ALL=(ALL:ALL) NOPASSWD: ALL
%IT ALL=(ALL) NOPASSWD:/bin/mkdir, PASSWD:/bin/rm
%www-data ALL=(ALL:ALL) NOPASSWD:/usr/sbin/service apache2 *
```

- `armour` can run ARMOUR_CMD commands with password prompt
- `Rahul` can run all commands without password prompt
- `IT` group can run mkdir without password, but rm requires password
- `www-data` can run apache2 service commands as any user without password

> [!IMPORTANT]
> `Rahul ALL=(ALL:ALL) NOPASSWD: ALL` is effectively an unauthenticated root grant. Compromise of the `Rahul` account — or any process running as it, including a web app — becomes instant, silent root. Rules like this are a top finding in privilege-escalation reviews; avoid `NOPASSWD: ALL` entirely.

## Best Practices

- Use `NOPASSWD` sparingly for commands that are safe and have limited potential impact.
- Avoid `NOPASSWD` on highly sensitive commands to prevent unauthorized privilege escalation.
- Combine with command aliases to limit scope strictly.
- Always test sudoers configurations with `visudo` before deploying.

> [!TIP]
> Additional hardening for `NOPASSWD` rules:
> - Pin exact command **paths and argument patterns**; a bare wildcard (`*`) often permits an unintended breakout.
> - Prefer running the automated task under a dedicated service account with a single narrowly scoped rule, rather than granting `NOPASSWD` to interactive users.
> - Cross-check every `NOPASSWD` command against GTFOBins to confirm it cannot spawn a shell, write arbitrary files, or read privileged data.

## Security Considerations

`NOPASSWD` rules are among the highest-value targets in a Linux privilege-escalation assessment because they convert a low-privileged foothold directly into root without any credential. When auditing a host, enumerate them from every reachable account:

```bash
sudo -l
```

Then confirm each listed command is truly non-escalatable. CIS and NIST hardening guidance both recommend minimizing passwordless sudo and reviewing `/etc/sudoers.d/*` for stray `NOPASSWD` entries introduced by packages or automation.

## Checking Effective Permissions

After your sudoers configuration, users can verify their allowed commands and password requirements via:

```bash
sudo -l
```

This lists commands user can run; it will show which ones require a password and which ones are `NOPASSWD`.

## Related

- [Sudo](Sudo.md) — parent sudo command and policy model.
- [Command-Aliases-in-sudoers](Command-Aliases-in-sudoers.md) — NOPASSWD applied to command aliases.
- [User-Aliases-in-sudoers](User-Aliases-in-sudoers.md) — NOPASSWD scoped to user aliases.
- Privilege-Escalation — NOPASSWD entries as a privilege-escalation vector.
- [Linux Administration & Server Hardening](../Readme.md) — course hub.
