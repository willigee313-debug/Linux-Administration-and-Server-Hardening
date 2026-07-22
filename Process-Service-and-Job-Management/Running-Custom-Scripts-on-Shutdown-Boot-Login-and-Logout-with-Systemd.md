# Running Custom Scripts on Shutdown, Boot, Login, and Logout with Systemd

## Overview

`systemd` lets you hook custom scripts into key system lifecycle events — **boot**, **shutdown**, **login**, and **logout** — by wrapping them in *unit* files. Instead of the old `rc.local` or scattered init scripts, you declare a small service unit, tell systemd *when* it should run relative to a target (for example `Before=shutdown.target` or `After=network.target`), and enable it.

This note walks through creating the scripts, writing the matching `oneshot` service units, enabling them, and the two logout paths (shell `~/.bash_logout` versus user-level systemd). The same mechanics that make systemd hooks convenient for administrators also make them a persistence and privilege-escalation target, so the placement and permissions of these files matter.

> [!NOTE]
> A `Type=oneshot` service runs a command to completion and then considers itself "done" — ideal for fire-and-forget lifecycle scripts, as opposed to a long-running daemon.

## Concepts

### Lifecycle Events and Where They Hook

```mermaid
flowchart LR
    subgraph Boot
        A[Kernel + systemd PID 1] --> B[network.target]
        B --> C[multi-user.target]
        C --> D[graphical.target]
    end
    D --> E[User login]
    E --> F[Interactive session]
    F --> G[User logout]
    G --> H[shutdown.target]

    B -.After.-> S1[run-after-poweron.service]
    D -.After.-> S2[run-after-user-login.service]
    F -.~/.bash_logout.-> S3[shell logout script]
    H -.Before.-> S4[run-before-shutdown.service]
```

### Event-to-Mechanism Map

| Event | Mechanism | Unit directive that anchors timing |
|-------|-----------|------------------------------------|
| Boot | System service unit | `After=network.target`, `WantedBy=multi-user.target` |
| Shutdown | System service unit | `Before=shutdown.target`, `WantedBy=shutdown.target` |
| Graphical login | System (or user) service unit | `After=graphical.target` |
| Shell logout | `~/.bash_logout` | Runs on interactive shell exit |
| Per-user login | User-level systemd unit | `~/.config/systemd/user/` |

## 1. Create Sample Scripts

```bash
vim /opt/shutdown_notify.sh
```

```bash
#!/bin/bash
echo "System shutting down at $(date)" >> /opt/shutdown.log
```

```bash
vim /opt/poweron_notify.sh
```

```bash
#!/bin/bash
echo "System started at $(date)" >> /opt/poweron.log
```

- Make them executable:

```bash
chmod +x /opt/shutdown_notify.sh
```

```bash
chmod +x /opt/poweron_notify.sh
```

## 2. Create Systemd Services

### Run Script Before Shutdown

```bash
vim /etc/systemd/system/run-before-shutdown.service
```

```ini
[Unit]
Description=Run custom script before shutdown
DefaultDependencies=no
Before=shutdown.target

[Service]
Type=oneshot
ExecStart=/opt/shutdown_notify.sh
TimeoutStartSec=0

[Install]
WantedBy=shutdown.target
```

- `DefaultDependencies=no` → ensures it runs even late in shutdown.
- `Before=shutdown.target` → guarantees it runs **before shutdown finalizes**.

### Run Script At Boot (After Network)

```bash
vim /etc/systemd/system/run-after-poweron.service
```

```ini
[Unit]
Description=Run custom script at boot
After=network.target

[Service]
Type=oneshot
ExecStart=/opt/poweron_notify.sh

[Install]
WantedBy=multi-user.target
```

- Runs **once at boot**, after the network is ready.
- Use `multi-user.target` for non-graphical systems (or `graphical.target` if you need the GUI up).

### Run Script After User Login (Graphical)

```bash
vim /etc/systemd/system/run-after-user-login.service
```

```ini
[Unit]
Description=Run custom script after graphical login
After=graphical.target

[Service]
Type=oneshot
ExecStart=/opt/poweron_notify.sh

[Install]
WantedBy=graphical.target
```

- Runs when the graphical environment is available.
- For **per-user login scripts**, prefer user-level systemd (see below).

## 3. Enable the Services

```bash
systemctl daemon-reload
```

```bash
systemctl enable run-before-shutdown.service
```

```bash
systemctl enable run-after-poweron.service
```

```bash
systemctl enable run-after-user-login.service
```

Reboot to test.

## 4. Run a Script on User Logout

For **shell logout** (e.g., an SSH or terminal session), edit `~/.bash_logout`:

```bash
#!/bin/bash
echo "User $(whoami) logged out at $(date)" >> /opt/logout.log
```

Make it executable:

```bash
chmod +x ~/.bash_logout
```

> [!NOTE]
> `.bash_logout` only works for **interactive login shells**, not GUI logouts. It also does not run for non-interactive shells or when a session is killed abruptly.

## 5. User-Level Systemd (Alternative Approach)

Instead of system-wide units, you can create **user services** that run without root and only for your account.

- Place files in:

```bash
~/.config/systemd/user/
```

Example — `~/.config/systemd/user/run-at-login.service`:

```ini
[Unit]
Description=Run script at user login

[Service]
Type=oneshot
ExecStart=/opt/poweron_notify.sh
```

Enable it:

```bash
systemctl --user daemon-reload
systemctl --user enable run-at-login.service
```

This runs **only for your user**, without requiring root.

> [!TIP]
> For user services to run when the user is not logged in (or to keep running after logout), enable *lingering* for that account: `loginctl enable-linger <user>`.

## Summary Table

| Event | Where to Configure | Example Unit / Script |
|---|---|---|
| **Shutdown** | systemd service | `run-before-shutdown.service` |
| **Boot** | systemd service | `run-after-poweron.service` |
| **User Login** | systemd service | `run-after-user-login.service` (or user-level unit) |
| **User Logout** | `~/.bash_logout` | Custom logout script |

## Troubleshooting

| Symptom | Likely Cause | First Check |
|---------|--------------|-------------|
| Service does nothing at boot | Not enabled or wrong target | `systemctl is-enabled <svc>`; `systemctl status <svc>` |
| Edited a unit but no change | Daemon not reloaded | `systemctl daemon-reload` |
| Shutdown script skipped | Runs too late / dependency ordering | Add `DefaultDependencies=no` and `Before=shutdown.target` |
| `~/.bash_logout` never runs | Non-interactive or GUI session | Confirm it's an interactive login shell |
| User unit inactive when logged out | Lingering not enabled | `loginctl enable-linger <user>` |

## Security Considerations

- Scripts referenced by boot/shutdown/login units typically run as **root**. If the script (e.g. `/opt/poweron_notify.sh`) or its directory is **writable by a non-root user**, that user can escalate to root by editing the script — keep them `root:root` and non-world-writable.
- `/etc/systemd/system/` unit files themselves must not be writable by unprivileged users; a modifiable `ExecStart=` is a direct privilege-escalation and persistence vector.
- Attacker-planted lifecycle units (or a tampered `~/.bash_logout`) are a common persistence mechanism. During incident response, review `systemctl list-unit-files --state=enabled`, the contents of `/etc/systemd/system/`, and each user's `~/.config/systemd/user/` and shell startup/logout files.
- Prefer user-level units (`systemctl --user`) for per-user automation so scripts run with the user's own privileges rather than root.

## References

- `man systemd.unit`, `man systemd.service`, `man systemd.special`, `man systemctl`, `man loginctl`
- `man bash` (see the *INVOCATION* section for `~/.bash_logout` behavior)

## Related

- [Service-Management-in-Linux](Service-Management-in-Linux.md) — manage systemd units with `systemctl` and `journalctl`.
- [Systemd-Timers-in-Linux](Systemd-Timers-in-Linux.md) — schedule systemd unit execution on a timer.
- [Cron-Jobs-in-Linux](Cron-Jobs-in-Linux.md) — alternative mechanism for scheduled scripting.
- [Process-Management-in-Linux](Process-Management-in-Linux.md) — inspect and control the processes these units spawn.
- Privilege-Escalation — writable boot/login scripts and units abused for root.
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub.
