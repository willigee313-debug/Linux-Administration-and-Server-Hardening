# Monitoring

## Overview

Observability is what turns a Linux server from a black box into a system you can actually operate: without logs, metrics, and alerts, an outage is discovered by a user complaint rather than a page. This module covers the full stack — kernel and application logging with [System-Logging-with-journald](System-Logging-with-journald.md) and [Logging-with-rsyslog](Logging-with-rsyslog.md), log lifecycle management via [Log-Rotation-with-logrotate](Log-Rotation-with-logrotate.md), security event auditing with [Auditing-with-auditd](Auditing-with-auditd.md), and the modern metrics/visualization/alerting pipeline built on [Metrics-with-Prometheus](Metrics-with-Prometheus.md), [Dashboards-with-Grafana](Dashboards-with-Grafana.md), and [Alerting-with-Alertmanager](Alerting-with-Alertmanager.md). It closes with real-time and infrastructure-wide monitoring tools and centralized log aggregation for multi-host estates.

> [!NOTE]
> **Observability has three legs — logs (what happened), metrics (how the system is behaving over time), and alerts (someone needs to act now). A hardened server needs all three; skipping any one leaves a blind spot.**

## Learning Objectives

- Read, filter, and persist logs with `journald` and route them with `rsyslog`, including remote log forwarding.
- Configure `logrotate` policies to prevent disks from filling up with unbounded log growth.
- Deploy `auditd` to generate tamper-evident, CIS/NIST-aligned audit trails for compliance and incident response.
- Stand up a Prometheus + node_exporter metrics pipeline and query it with PromQL.
- Build operational dashboards in Grafana and wire Prometheus alert rules through Alertmanager to notification channels.
- Compare lightweight real-time dashboards (Netdata) against traditional agent-based infrastructure monitoring (Nagios/Zabbix) and lightweight service watchdogs (Monit).
- Design a centralized logging architecture (e.g., rsyslog/syslog-ng forwarders or an ELK/Loki-style pipeline) for a fleet of servers.

## Topics Covered

| Note | What it covers |
|---|---|
| [System-Logging-with-journald](System-Logging-with-journald.md) | systemd's structured binary journal: `journalctl` filtering, persistence, retention, and journal-to-syslog forwarding |
| [Logging-with-rsyslog](Logging-with-rsyslog.md) | Traditional syslog daemon: facilities/priorities, rule syntax, local file routing, and remote log shipping over TCP/TLS |
| [Log-Rotation-with-logrotate](Log-Rotation-with-logrotate.md) | Rotation policies, compression, retention windows, and custom `postrotate` scripts to avoid disk exhaustion |
| [Auditing-with-auditd](Auditing-with-auditd.md) | Linux Audit Framework: audit rules, watching files/syscalls, `ausearch`/`aureport`, and compliance-driven audit policy |
| [Metrics-with-Prometheus](Metrics-with-Prometheus.md) | Pull-based metrics collection, `node_exporter`, scrape configs, PromQL, and service discovery |
| [Dashboards-with-Grafana](Dashboards-with-Grafana.md) | Data source wiring, dashboard/panel design, variables, and provisioning dashboards as code |
| [Alerting-with-Alertmanager](Alerting-with-Alertmanager.md) | Alert rule design, routing trees, grouping/inhibition, silences, and notification receivers (email, Slack, PagerDuty) |
| [Real-Time-Monitoring-with-Netdata](Real-Time-Monitoring-with-Netdata.md) | Zero-config, per-second real-time system monitoring with an embedded web dashboard |
| [Infrastructure-Monitoring-with-Nagios-and-Zabbix](Infrastructure-Monitoring-with-Nagios-and-Zabbix.md) | Agent/plugin-based host and service monitoring, checks, and escalation for larger fleets |
| [Service-Monitoring-with-Monit](Service-Monitoring-with-Monit.md) | Lightweight process/service watchdog with automatic restart actions and simple alerting |
| [Centralized-Logging](Centralized-Logging.md) | Aggregating logs from many hosts into a single searchable store; forwarder patterns and retention strategy |

## Practical Labs

1. **Log pipeline build-out**: Configure `rsyslog` on a client to forward all logs to a central rsyslog server over TLS, then set a `logrotate` policy on the receiver to compress logs older than 1 day and purge after 30 days.
2. **Metrics-to-alert loop**: Install `node_exporter` on two VMs, scrape them with Prometheus, build a Grafana dashboard showing CPU/memory/disk, then create an Alertmanager rule that fires (and routes to a test webhook) when disk usage exceeds 85%.
3. **Audit trail for privilege use**: Configure `auditd` rules to watch `/etc/passwd`, `/etc/shadow`, and all `execve` calls by UID 0; trigger a change and use `ausearch`/`aureport` to reconstruct exactly what happened.

## Best Practices

- Centralize logs off-host immediately — a compromised or crashed server's local logs are the first thing an attacker deletes.
- Set explicit retention on every log source (`journald` `SystemMaxUse=`, `logrotate` `rotate N`) — unbounded logs are a disk-exhaustion DoS waiting to happen.
- Alert on symptoms (latency, error rate, saturation) rather than every possible cause; noisy alerting trains responders to ignore pages.
- Version-control Prometheus scrape configs, alert rules, and Grafana dashboard JSON — treat observability config as infrastructure code.
- Use PromQL recording rules for expensive queries reused across dashboards/alerts instead of recomputing them ad hoc.
- Keep monitoring agents (node_exporter, Zabbix agent, Netdata) patched and bound to least-privilege scrape endpoints.

## Security Considerations

- **CIS Controls / NIST SP 800-53 AU family** (Audit and Accountability) map directly to `auditd` and centralized logging — treat audit log integrity as a compliance requirement, not an option.
- Protect log integrity: ship logs to a write-once or access-restricted central store so on-host tampering doesn't erase evidence (aligns with CIS Benchmark recommendations to forward `auth.log`/`secure` off-host).
- Restrict `journalctl`, `/var/log/audit/`, and Grafana/Prometheus admin access to authorized accounts only — logs and dashboards often expose internal topology and secrets in query strings.
- Encrypt log transport (rsyslog over TLS, or a VPN/private network) — plaintext syslog over UDP/TCP 514 leaks sensitive data and is trivially spoofable.
- Set `auditd` to fail closed (`-f 2` panic on audit failure) on systems with strict compliance requirements, per CIS RHEL benchmarks.
- Avoid exposing Prometheus, Grafana, or Netdata dashboards directly to the internet without authentication — these have historically been scanned and abused for internal reconnaissance.

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `journalctl` shows no logs after reboot | Journal is volatile (`/run/log/journal`) | Set `Storage=persistent` in `/etc/systemd/journald.conf` and create `/var/log/journal` |
| Remote rsyslog messages not arriving | Firewall blocking 514/TCP or TLS cert mismatch | Check `firewalld`/`ufw` rules and verify `$DefaultNetstreamDriverCAFile` paths |
| Disk fills up despite logrotate config | Wrong glob path or service holds file handle open | Verify `logrotate -d` dry-run output; add `copytruncate` for services that don't reopen log files |
| Prometheus target shows `DOWN` | Firewall, wrong port, or exporter crashed | `curl http://host:9100/metrics` from the Prometheus host to isolate network vs. exporter issue |
| Alertmanager not sending notifications | Route tree misconfigured or receiver webhook unreachable | Check `amtool config routes test` and Alertmanager logs for delivery errors |
| `auditd` rules not persisting after reboot | Rules loaded via `auditctl` only, not `/etc/audit/rules.d/` | Add rules to `/etc/audit/rules.d/*.rules` and run `augenrules --load` |

> [!NOTE]
> **📸 Screenshot**
> _Capture: A Grafana dashboard panel showing node_exporter CPU/memory/disk metrics alongside an active Alertmanager alert in the Grafana alerting view._

## References

- [systemd journald documentation](https://www.freedesktop.org/software/systemd/man/journald.conf.html)
- [rsyslog official documentation](https://www.rsyslog.com/doc/)
- [Prometheus documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana documentation](https://grafana.com/docs/grafana/latest/)
- [Linux Audit Framework (auditd) documentation](https://man7.org/linux/man-pages/man8/auditd.8.html)
- [CIS Benchmarks for Linux](https://www.cisecurity.org/cis-benchmarks)

## Related Notes

- [System-Logging-with-journald](System-Logging-with-journald.md) — System Logging with journald
- [Logging-with-rsyslog](Logging-with-rsyslog.md) — Logging with rsyslog
- [Log-Rotation-with-logrotate](Log-Rotation-with-logrotate.md) — Log Rotation with logrotate
- [Auditing-with-auditd](Auditing-with-auditd.md) — Auditing with auditd
- [Metrics-with-Prometheus](Metrics-with-Prometheus.md) — Metrics with Prometheus
- [Dashboards-with-Grafana](Dashboards-with-Grafana.md) — Dashboards with Grafana
- [Alerting-with-Alertmanager](Alerting-with-Alertmanager.md) — Alerting with Alertmanager
- [Real-Time-Monitoring-with-Netdata](Real-Time-Monitoring-with-Netdata.md) — Real-Time Monitoring with Netdata
- [Infrastructure-Monitoring-with-Nagios-and-Zabbix](Infrastructure-Monitoring-with-Nagios-and-Zabbix.md) — Infrastructure Monitoring with Nagios and Zabbix
- [Service-Monitoring-with-Monit](Service-Monitoring-with-Monit.md) — Service Monitoring with Monit
- [Centralized-Logging](Centralized-Logging.md) — Centralized Logging
- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
