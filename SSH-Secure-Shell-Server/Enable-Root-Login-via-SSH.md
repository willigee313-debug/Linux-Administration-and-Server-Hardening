# Enable Root Login via SSH

## Overview

The `PermitRootLogin` directive in `sshd_config` governs whether the `root` account may authenticate directly over SSH. Modern OpenSSH ships with this **disabled** (`prohibit-password` or `no`) because direct root login removes accountability and hands an attacker the highest-value account in a single successful authentication.

> [!WARNING]
> Enabling root login is **risky** and is discouraged in production. Only do this in secure, trusted, or isolated environments (labs, VMs, disposable test hosts). In production, log in as an unprivileged user and escalate with `sudo` — see [Prevent-Root-Login-via-SSH](Prevent-Root-Login-via-SSH.md) for the recommended hardened state.

This note documents how to enable it deliberately (and how to verify no conflicting drop-in silently overrides your setting), for the narrow cases where it is justified.

## Concepts

| `PermitRootLogin` value | Behaviour |
|-------------------------|-----------|
| `yes` | Root may log in with password **or** key |
| `prohibit-password` | Root may log in with **keys only** (no password) — the safer middle ground |
| `forced-commands-only` | Root may log in only to run a forced command (e.g., automated backups) |
| `no` | Root SSH login is fully disabled (recommended default) |

> [!TIP]
> If you have a genuine need for automated root access (backups, orchestration), prefer `prohibit-password` with a dedicated key over `yes`. It preserves key-only security while allowing the workflow.

## Configuration

OpenSSH reads the main file first, then applies files matched by the `Include /etc/ssh/sshd_config.d/*.conf` line. Because of `Include` ordering, a drop-in can override the main file — this is the most common reason a `PermitRootLogin` change appears to "not work".

### Option 1: Main Config File

Edit the main SSH config:

```bash
vim /etc/ssh/sshd_config
```

Ensure this line exists **and is not commented**:

```bash
PermitRootLogin yes
```

### Option 2: Drop-in Config

Alternatively, create or edit a drop-in config file:

```bash
vim /etc/ssh/sshd_config.d/01.permitrootlogin.conf
```

Set:

```bash
PermitRootLogin yes
```

> This will override any conflicting setting from the main config, because drop-ins in `sshd_config.d/` are included and the **first** matching value in the parse order wins.

### Check For Conflicts

Make sure no other file still sets it to `no`:

```bash
grep -r "PermitRootLogin" /etc/ssh/sshd_config*
```

> Comment or remove any lines like `PermitRootLogin no`.

## Commands

### Restart SSH

Apply the changes:

```bash
systemctl restart sshd.service
```

### Test Root Login

Try logging in as root (use your custom port if configured):

```bash
ssh -p 2200 root@192.168.1.32
```

> If successful, the shell should drop you into a root session.

> [!NOTE]
> **📸 Screenshot**
> _Capture: Terminal showing a successful SSH login as root with the shell prompt ending in a hash symbol_

## Security Considerations

- If `PasswordAuthentication no` is enabled, root login will **still fail** unless you use key-based authentication. Combine `PermitRootLogin yes` with a valid key in `/root/.ssh/authorized_keys`, or set `PermitRootLogin prohibit-password`.
- On SELinux systems, make sure the port is allowed for SSH:

```bash
semanage port -l | grep ssh
```

- If using firewalld, make sure your port (e.g., 2222) is open:

```bash
firewall-cmd --permanent --add-port=2200/tcp
firewall-cmd --reload
```

> [!IMPORTANT]
> Direct root login defeats per-user accountability — commands in the audit log show only "root", not who ran them. Every hardening standard (CIS, NIST 800-53, DISA STIG) mandates `PermitRootLogin no` on production systems. Revert to that state when the lab task is done: [Prevent-Root-Login-via-SSH](Prevent-Root-Login-via-SSH.md).

## Best Practices

- Prefer **`sudo` from a named account** over enabling root SSH; it preserves attribution.
- If root SSH is unavoidable, use **`prohibit-password`** plus a hardware-backed or passphrase-protected key.
- Restrict source IPs with `AllowUsers root@<trusted-ip>` ([Managing-IP-Allow-and-Deny-in-SSH](Managing-IP-Allow-and-Deny-in-SSH.md)) so root cannot be targeted from arbitrary hosts.
- Always validate config with `sshd -t` before restarting and keep a fallback session open.

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---------|--------------|------------|
| Root login still refused after `yes` | A drop-in sets `no` and wins the parse order | `grep -r PermitRootLogin /etc/ssh/sshd_config*` and remove it |
| "Permission denied" with correct password | `PasswordAuthentication no` is set | Use key auth or enable password auth |
| Change ignored entirely | `sshd` not restarted, or syntax error | Run `sshd -t`, then `systemctl restart sshd.service` |
| Works but SELinux denies on custom port | Port not labelled `ssh_port_t` | `semanage port -a -t ssh_port_t -p tcp <port>` |

## References

- `man 5 sshd_config` — `PermitRootLogin`, `PasswordAuthentication`
- CIS Benchmarks / DISA STIG — "Disable SSH root login"

## Related

- [Prevent-Root-Login-via-SSH](Prevent-Root-Login-via-SSH.md) — opposite (recommended) hardening control
- [SSH(Secure-Shell)-Server](SSH(Secure-Shell)-Server.md) — parent SSH server note
- [Managing-IP-Allow-and-Deny-in-SSH](Managing-IP-Allow-and-Deny-in-SSH.md) — restrict who and where root may connect from
- SSH-Enumeration — attacker-side SSH recon
- Privilege-Escalation — root SSH access aids escalation
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
