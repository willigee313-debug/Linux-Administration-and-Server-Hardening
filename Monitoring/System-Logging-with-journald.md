# System Logging with journald

## Overview

`systemd-journald` is the logging service built into systemd that collects and indexes kernel, service, and syslog messages into a structured, binary journal. Every unit managed by systemd — see [Service-Management-in-Linux](../Process-Service-and-Job-Management/Service-Management-in-Linux.md) — has its stdout/stderr captured automatically, which makes journald the first place to look when a service fails to start. It commonly runs alongside [Logging-with-rsyslog](Logging-with-rsyslog.md) (journald forwards to it for traditional flat-file logs) and is complemented by [Log-Rotation-with-logrotate](Log-Rotation-with-logrotate.md) for rsyslog's own text logs, since the journal manages its own retention independently.

> [!NOTE]
> **Structured logging by default**
> Because the journal stores messages as indexed binary records with metadata (unit, PID, UID, boot ID, priority), `journalctl` can filter with precision that grepping flat text files cannot match — filter by service, boot, time range, or severity in a single command.

## Concepts

- **Journal**: an append-only, binary, indexed log store. Each entry carries structured fields (`_SYSTEMD_UNIT`, `_PID`, `_UID`, `_BOOT_ID`, `PRIORITY`, `MESSAGE`, etc.).
- **Volatile vs. persistent storage**: by default the journal lives in `/run/log/journal/` (tmpfs, cleared on reboot) unless persistent storage is enabled, which moves it to `/var/log/journal/`.
- **Boot ID**: a unique identifier per boot, letting you scope queries to "this boot" (`-b`) or a prior one (`-b -1`).
- **Priority levels**: syslog-compatible severities `0` (emerg) through `7` (debug) — `emerg, alert, crit, err, warning, notice, info, debug`.
- **Journal forwarding**: journald can forward entries to syslog (rsyslog), the kernel log buffer (`kmsg`), and the console, controlled independently of local storage.
- **Catalog messages**: some entries carry a `MESSAGE_ID` that maps to an explanatory catalog entry (`journalctl -x`).

## Architecture

```mermaid
flowchart LR
    A[Kernel ring buffer] --> J[systemd-journald]
    B[systemd units\nstdout/stderr] --> J
    C[/dev/log\nsyslog socket/] --> J
    D[Native journal API\nsd_journal_send] --> J
    J -->|Storage=persistent| E[(/var/log/journal/)]
    J -->|Storage=volatile default| F[(/run/log/journal/)]
    J -->|ForwardToSyslog=yes| G[rsyslog]
    G --> H[(/var/log/*.log)]
    J -->|ForwardToConsole=yes| I[/dev/console]
```

## Installation

`systemd-journald` ships with systemd on virtually every modern distribution and requires no separate installation. Confirm it is active and check which package provides the CLI tools.

```bash
# Verify journald is running (RHEL/Debian family alike)
systemctl status systemd-journald

# journalctl is part of the systemd package
dpkg -S "$(command -v journalctl)"     # Debian/Ubuntu
rpm -qf "$(command -v journalctl)"     # RHEL/Fedora/CentOS
```

## Configuration

Main configuration file: `/etc/systemd/journald.conf` (drop-ins in `/etc/systemd/journald.conf.d/*.conf` are preferred so package updates don't clobber edits).

```ini
# /etc/systemd/journald.conf.d/00-hardening.conf
[Journal]
Storage=persistent
Compress=yes
SystemMaxUse=1G
SystemKeepFree=500M
SystemMaxFileSize=100M
MaxRetentionSec=1month
MaxFileSec=1week
ForwardToSyslog=yes
ForwardToConsole=no
RateLimitIntervalSec=30s
RateLimitBurst=10000
```

| Directive | Purpose |
|---|---|
| `Storage=` | `volatile` (tmpfs, default), `persistent` (`/var/log/journal/`), `auto`, or `none` |
| `Compress=` | Compress large journal fields (default `yes`) |
| `SystemMaxUse=` | Cap total disk space the journal may consume |
| `SystemKeepFree=` | Minimum free disk space to always leave available |
| `SystemMaxFileSize=` | Max size of an individual rotated journal file |
| `MaxRetentionSec=` / `MaxFileSec=` | Time-based retention / rotation interval |
| `ForwardToSyslog=` | Forward entries to the local syslog socket (rsyslog/syslog-ng) |
| `ForwardToKMsg=` | Forward to the kernel log buffer |
| `ForwardToConsole=` | Forward to a TTY (`TTYPath=`, default `/dev/console`) |
| `RateLimitIntervalSec=` / `RateLimitBurst=` | Throttle noisy services to prevent log flooding |

Apply changes and enable persistent storage:

```bash
# Option A: let journald manage the directory
sudo systemctl edit systemd-journald    # or drop a file in journald.conf.d/
sudo systemctl restart systemd-journald

# Option B: manually create the persistent directory (common bootstrap step)
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
sudo systemctl restart systemd-journald
```

> [!TIP]
> **Persistent storage after a fresh install**
> Many distros ship with `Storage=auto` and no `/var/log/journal/` directory, so logs vanish on reboot. Creating the directory (as above) is enough to trigger persistent storage without editing the config file at all.

## Commands

| Command | Purpose |
|---|---|
| `journalctl` | Show the full journal, oldest first, paged |
| `journalctl -u sshd` | Filter by systemd unit |
| `journalctl -u sshd -u nginx` | Filter by multiple units |
| `journalctl -p err` | Filter by minimum priority (`err` and higher) |
| `journalctl -p warning..err` | Filter by a priority range |
| `journalctl --since "2026-07-20" --until "2026-07-21 09:00"` | Filter by absolute time range |
| `journalctl --since "1 hour ago"` | Filter by relative time |
| `journalctl -f` | Follow mode, like `tail -f` |
| `journalctl -f -u nginx` | Follow a specific unit's log live |
| `journalctl -b` | Logs from the current boot only |
| `journalctl -b -1` | Logs from the previous boot |
| `journalctl --list-boots` | List all known boot IDs |
| `journalctl -k` | Kernel messages only (like `dmesg`) |
| `journalctl _PID=1234` | Filter by an arbitrary structured field |
| `journalctl -o json-pretty` | Structured JSON output |
| `journalctl -x` | Add explanatory context from the message catalog |
| `journalctl --disk-usage` | Show how much disk space the journal is using |
| `journalctl --vacuum-size=500M` | Shrink the journal to a target size |
| `journalctl --vacuum-time=2weeks` | Delete entries older than a duration |
| `journalctl -n 50` | Show the last N lines |

## Examples

Combine filters — this is where journald shines over flat-file `grep`:

```bash
# Errors from sshd in the last hour, following live
journalctl -u sshd -p err --since "1 hour ago" -f

# Everything from the current boot, priority warning or worse
journalctl -b -p warning

# All output from a specific process ID across any unit
journalctl _PID=$(pgrep -x nginx | head -1)

# Export a time-boxed slice for incident analysis
journalctl --since "2026-07-21 22:00" --until "2026-07-22 02:00" -o json > incident.json

# Confirm why a unit failed to start
journalctl -u myapp.service -b --no-pager | tail -n 40
```

## Best Practices

- Enable `Storage=persistent` on any host you care about surviving a reboot for forensic or audit purposes.
- Set `SystemMaxUse=` and `SystemKeepFree=` explicitly — an unbounded journal can fill `/var` and take down the host.
- Prefer drop-in files under `journald.conf.d/` over editing `journald.conf` directly, so config survives package upgrades.
- Use `-u` and `--since`/`--until` together instead of piping through `grep` — structured filtering is faster and more precise on large journals.
- When forwarding to rsyslog for long-term retention or centralized shipping, confirm `ForwardToSyslog=yes` and that rsyslog's `imjournal` (RHEL) or journal input is enabled — see [Logging-with-rsyslog](Logging-with-rsyslog.md).
- Run `journalctl --vacuum-size=` or `--vacuum-time=` on a schedule (cron/systemd timer) on space-constrained hosts if you don't already cap size in config.
- Use `journalctl -o json` output when piping into log analysis tools or SIEM ingestion for reliable field parsing.

## Security Considerations

- **Restrict journal access**: journal files are readable by the `systemd-journal` group; only add trusted admin accounts to that group (CIS control: minimize local log access).
- **Tamper resistance**: enable `Seal=yes` (Forward Secure Sealing, if supported) to detect retroactive tampering with journal files — relevant for audit integrity requirements (NIST SI-4, AU-9).
- **Rate limiting**: keep `RateLimitIntervalSec`/`RateLimitBurst` tuned so a compromised or malfunctioning service can't flood the journal and evict evidence of an earlier attack via rotation.
- **Centralize critical logs**: local persistent storage alone is insufficient for incident response if the host itself is compromised — forward to rsyslog and a remote/central log collector (satisfies audit log protection controls, CIS 4.2.1.x / NIST AU-4).
- **Retention alignment**: set `MaxRetentionSec=` to match your organization's log retention policy, not just disk-space convenience.
- **Audit `journalctl` access**: since journald exposes kernel and auth-related messages, treat read access to `/var/log/journal/` with the same sensitivity as `/var/log/secure` or `/var/log/auth.log`.

> [!WARNING]
> **Volatile storage loses evidence**
> On systems left at the default `Storage=auto` with no `/var/log/journal/` directory, a reboot after a compromise silently destroys journal evidence. Enable persistent storage and forward to a remote collector before you need it, not after.

## Troubleshooting

- **`journalctl` shows nothing for a unit**: confirm the unit name is exact (`systemctl status <unit>` shows the canonical name) and that you aren't scoping to the wrong boot with `-b`.
- **Journal grows unbounded**: check `journalctl --disk-usage`; set `SystemMaxUse=`/`SystemKeepFree=` and run `--vacuum-size=`.
- **Logs disappear after reboot**: `Storage=` is still `volatile`/`auto` with no persistent directory — create `/var/log/journal/` as shown above.
- **No forwarding to `/var/log/messages` or `/var/log/syslog`**: verify `ForwardToSyslog=yes` in journald.conf and that rsyslog's journal import module is loaded (`imjournal` on RHEL, `imuxsock`/journal socket on Debian).
- **`Failed to open system journal`**: check permissions on `/var/log/journal/<machine-id>/` and disk space with `df -h /var/log`.
- **High CPU from journald**: often caused by a service logging excessively; identify the noisy unit with `journalctl -u <unit> | wc -l` per candidate, then apply per-unit rate limiting or fix the application.

> [!NOTE]
> **📸 Screenshot**
> _Capture: terminal output of `journalctl -u sshd -p err --since "1 hour ago"` showing filtered, timestamped entries_

## References

- `man journald.conf`
- `man journalctl`
- `man systemd-journald.service`
- freedesktop.org systemd documentation: https://www.freedesktop.org/software/systemd/man/journald.conf.html

## Related Notes

- [Logging-with-rsyslog](Logging-with-rsyslog.md) — forwarding journald entries to traditional flat-file syslog
- [Log-Rotation-with-logrotate](Log-Rotation-with-logrotate.md) — rotating rsyslog's flat-file logs (journald rotates itself)
- [Service-Management-in-Linux](../Process-Service-and-Job-Management/Service-Management-in-Linux.md) — the systemd units whose output journald captures
- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
