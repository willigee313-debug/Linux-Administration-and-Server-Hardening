# SELinux Booleans and Ports

SELinux **booleans** are runtime on/off switches that toggle chunks of policy without recompiling it, and the **port type** mapping tells SELinux which port numbers a given service context is allowed to bind — together they are the two levers used most often to make a policy-compliant service work on a non-default configuration.

## Overview

| Aspect | Booleans | Ports |
|---|---|---|
| What they control | Optional policy branches (e.g. "can httpd reach the network?") | Which TCP/UDP ports a given SELinux port type may use |
| Query command | `getsebool -a` | `semanage port -l` |
| Change command | `setsebool [-P] <name> on\|off` | `semanage port -a/-m/-d -t <type> -p <proto> <port>` |
| Backing package | `policycoreutils` / `libselinux-utils` | `policycoreutils-python-utils` (RHEL) / `policycoreutils` (Debian) |
| Persistence | `-P` writes to policy store; without it, resets on reboot | `semanage port` changes are always persistent |

> [!NOTE]
> Booleans and ports solve **different classes of AVC denials**. A boolean denial mentions a policy capability (`httpd_can_network_connect`); a port denial mentions `name_bind` on an unlabeled or wrong-type port. `journalctl -t setroubleshoot` (or `ausearch -m avc`) usually tells you which one you are dealing with — see [SELinux-Troubleshooting](SELinux-Troubleshooting.md) for the full triage workflow.

## How It Works

Every confined process runs under a **domain type** (e.g. `httpd_t`). Policy grants that domain access to specific **object types** — files, sockets, and ports each carry their own type. A boolean is simply a named flag that the compiled policy checks before allowing an otherwise-denied rule to take effect; flipping it does not change any file on disk, it flips a bit the kernel's SELinux policy engine consults on every access decision. Ports work the same way but for network binds: `httpd_t` is allowed to `bind()`/`name_bind` only to ports labeled with an `http_port_t`-family type, and `semanage port` is how you add, move, or remove ports from that type's list.

```mermaid
flowchart TD
    A[Process wants an action] --> B{Type enforcement check}
    B -->|File/network object type has expected label| C[Boolean check: is optional rule enabled?]
    C -->|setsebool boolean = on| D[Allow]
    C -->|setsebool boolean = off| E[Deny -> AVC logged]
    B -->|Process tries to bind() a TCP/UDP port| F{Is port in this domain's allowed port type list?}
    F -->|Yes: semanage port -a mapped it| D
    F -->|No: unlabeled or wrong type| E
```

## Configuration: Booleans

### Step 1: List Current Booleans

`getsebool -a` shows every boolean and its current state; `semanage boolean -l` additionally shows the human-readable description from policy.

> Example:

```bash
getsebool -a | grep httpd
```

```bash
semanage boolean -l | grep httpd_can_network
```

```text
httpd_can_network_connect --> off      Allow HTTP daemon to connect to network...
httpd_can_network_connect_db --> off   Allow HTTP daemon to connect to database over network.
```

### Step 2: Toggle a Boolean at Runtime

`setsebool` alone changes the in-memory state only — it reverts on the next reboot. Use this to test before committing.

> Example:

```bash
setsebool httpd_can_network_connect on
```

```bash
getsebool httpd_can_network_connect
```

### Step 3: Make the Change Persistent

Add `-P` to write the change into the SELinux policy store so it survives a reboot (and a `restorecon`/relabel).

> Example:

```bash
setsebool -P httpd_can_network_connect on
```

> [!IMPORTANT]
> `setsebool -P` rebuilds part of the active policy module and can take a few seconds on a busy system — it is not instant like the non-`-P` form. Never script large batches of `-P` calls in a hot path; set them once during provisioning.

### Step 4: Query a Single Boolean's Description

```bash
semanage boolean -l | grep -i ftp_home_dir
```

## Common Booleans Reference

| Boolean | Effect when `on` | Typical use case |
|---|---|---|
| `httpd_can_network_connect` | Apache/nginx (`httpd_t`) may open outbound network connections | Reverse proxies, apps calling external APIs |
| `httpd_can_network_connect_db` | `httpd_t` may connect to a database over the network | Web app talking to a remote MySQL/PostgreSQL host |
| `httpd_can_sendmail` | `httpd_t` may send mail via `sendmail`/`postfix` | PHP `mail()` / contact forms |
| `httpd_enable_homedirs` | Apache may serve content from user home directories | `mod_userdir` (`~user/`) sites |
| `httpd_unified` | Treats all `httpd_*_content_t` types the same for read access | Simplifies labeling on dev boxes (not for production) |
| `ftpd_full_access` | `vsftpd`/`proftpd` (`ftpd_t`) gets full read/write and network access, bypassing most ftpd-specific restrictions | Internal FTP server needing broad filesystem access |
| `ftpd_connect_db` | FTP daemon may connect to a database over the network | FTP servers backed by SQL-based virtual users |
| `samba_export_all_rw` | Samba (`smbd_t`) may read/write **any** file matching its DAC permissions, ignoring the normal `samba_share_t` restriction | Sharing arbitrary directories outside `/srv/samba` |
| `samba_export_all_ro` | Same as above but read-only | Read-only exports of non-standard paths |
| `nis_enabled` | Allows services to make NIS/YP RPC calls | Legacy NIS authentication |
| `use_nfs_home_dirs` | Allows home directories to be served over NFS | NFS-mounted `/home` |

> [!WARNING]
> `httpd_can_network_connect` and `samba_export_all_rw` are broad, commonly-abused booleans. Enabling either turns off a real security boundary — if the web app or Samba daemon is later compromised, SELinux offers far less containment. Enable the **narrowest** boolean that solves the problem (e.g. `httpd_can_network_connect_db` instead of the blanket `httpd_can_network_connect`) and document why in change control.

## Configuration: Ports

### Step 1: List Current Port Mappings

`semanage port -l` dumps every SELinux port type and the port ranges bound to it. Filter for the type you care about.

> Example:

```bash
semanage port -l | grep http_port_t
```

```text
http_port_t                   tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
```

### Step 2: Add a Non-Standard Port to an Existing Type

If a web app listens on `8080/tcp` and you want `httpd_t` to be allowed to bind it, add `8080` to `http_port_t` rather than disabling enforcement.

> Example:

```bash
semanage port -a -t http_port_t -p tcp 8080
```

```bash
semanage port -l | grep 8080
```

### Step 3: Modify an Existing Custom Port Entry

Use `-m` (modify) instead of `-a` when the port is already mapped to a type and you need to change the protocol or add it to a range under the same command form.

> Example:

```bash
semanage port -m -t http_port_t -p tcp 8080
```

### Step 4: Remove a Custom Port Mapping

```bash
semanage port -d -t http_port_t -p tcp 8080
```

> [!NOTE]
> `semanage port -a` fails with "port already defined" if the port is already assigned to a **different** type — a common source of confusion. Check `semanage port -l | grep <port>` first; if it belongs to another type, either pick that type or `-d` it from the old type before adding it to the new one.

## Examples

### Debian / Ubuntu Note

CentOS Stream ships `policycoreutils-python-utils` (providing `semanage`, `setsebool`) by default when SELinux is installed; on Debian 12, SELinux is not the default MAC (AppArmor is) and must be installed explicitly:

```bash
apt install selinux-basics selinux-policy-default policycoreutils setools
```

```bash
selinux-activate
```

Once active, `getsebool`, `setsebool`, and `semanage boolean -l` / `semanage port -l` behave identically to RHEL-family systems — the userland tooling is the same upstream project.

### Verifying After a Reboot

Confirm a persistent boolean survived a reboot, and that a custom port survives a policy reload:

```bash
getsebool httpd_can_network_connect
```

```bash
semanage port -l | grep 8080
```

## Best Practices

- Prefer the **narrowest** boolean or port change that fixes the denial; avoid `httpd_unified`, `samba_export_all_rw`, and similar "disable most of the boundary" booleans in production.
- Always test with plain `setsebool` first, confirm the denial is gone, then commit with `-P` — don't jump straight to persistent changes.
- Add non-standard ports to the correct type with `semanage port -a` instead of disabling enforcement or running the service unconfined.
- Keep a record (change ticket, comment in a provisioning script) of every `setsebool -P` and `semanage port -a` so the deviation from stock policy is auditable.
- Use `semanage boolean -l` (not just `getsebool -a`) when documenting a change — it captures the policy's own description of what the boolean does.
- Bake required booleans/ports into your provisioning (Ansible, Kickstart `%post`, cloud-init) so a rebuilt host doesn't silently drift back to defaults.

## Security Considerations

Booleans and port mappings are precision tools; the failure mode to avoid is reaching for `setenforce 0` or `audit2allow -M` blanket modules instead.

> [!WARNING]
> Don't "fix" a boolean or port denial by disabling SELinux (`setenforce 0`) or generating an overly broad custom module with `audit2allow`. Both remove the containment SELinux provides for that service entirely, including for future, unrelated vulnerabilities. Diagnose the specific denial (`sealert -a /var/log/audit/audit.log`) and apply the narrowest boolean or port change that resolves it.

- Treat `httpd_can_network_connect`, `ftpd_full_access`, and `samba_export_all_rw` as high-impact changes requiring review — each significantly widens what a compromised service can do.
- Custom port mappings expand the attack surface a service's domain can reach; remove entries (`semanage port -d`) when a service is decommissioned or moved off a port.
- During an audit or incident response, diff `semanage boolean -l` and `semanage port -l` output against a known-good baseline to spot unauthorized policy loosening.
- Non-standard `-P` booleans and `semanage port` entries persist across reboots and updates — they are a legitimate persistence vector if an attacker gains root, since they can widen policy without touching enforcing mode at all.

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---|---|---|
| Service works, then breaks on reboot | Boolean was set without `-P` | Re-apply with `setsebool -P <bool> on` |
| `semanage port -a` fails: "port already defined" | Port already bound to a different SELinux type | `semanage port -l \| grep <port>`, then `-d` from old type or use `-m` on the correct type |
| App on custom port still gets `EACCES`/connection refused via SELinux | Port added to wrong type, or firewall (not SELinux) is blocking | Confirm with `semanage port -l \| grep <port>`; also check `firewalld`/`iptables` — see [Firewalld](Firewalld.md) |
| Denial persists after `setsebool -P` | Wrong boolean identified, or file context also wrong | Re-check `ausearch -m avc -ts recent`; verify with `getsebool` the exact name matches the denial |
| `semanage: command not found` | `policycoreutils-python-utils` (RHEL) not installed, or SELinux tooling absent on Debian | `dnf install policycoreutils-python-utils` / `apt install policycoreutils selinux-utils` |
| Boolean list is empty or `getsebool` errors | SELinux disabled or not installed | Check `getenforce`; see [SELinux-Fundamentals](SELinux-Fundamentals.md) for enabling/mode basics |

## References

- [SELinux Project — Booleans (Fedora/RHEL docs)](https://docs.fedoraproject.org/en-US/quick-docs/selinux-getting-started/)
- [Red Hat: Configuring SELinux Booleans](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/using_selinux/changing-selinux-states-and-modes_using-selinux)
- [Red Hat: Managing SELinux Port Contexts](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/using_selinux/troubleshooting-problems-related-to-selinux_using-selinux)
- `man semanage` / `man semanage-boolean` / `man semanage-port`
- `man setsebool` / `man getsebool`

## Related

- [SELinux-Fundamentals](SELinux-Fundamentals.md) — modes, contexts, and core concepts these tools build on
- [SELinux-Contexts-and-File-Labeling](SELinux-Contexts-and-File-Labeling.md) — file-side labeling that complements port/boolean policy
- [SELinux-Troubleshooting](SELinux-Troubleshooting.md) — AVC denial triage workflow (`ausearch`, `sealert`, `audit2allow`)
- [Firewalld](Firewalld.md) — network-layer filtering that works alongside SELinux port types
- [Security, Firewall & Monitoring](Readme.md) — module index
- [Linux Administration & Server Hardening](../Readme.md) — course hub
