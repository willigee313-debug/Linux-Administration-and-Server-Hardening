# Performance and Tuning

## Overview

Performance tuning is the discipline of measuring how a Linux system consumes CPU, memory, disk I/O, and network bandwidth, then adjusting workload placement or kernel parameters to remove bottlenecks before they become outages. This module builds a methodology around the classic USE (Utilization, Saturation, Errors) approach and ties it to the tools admins reach for daily — `top`, `vmstat`, `iostat`, `sar`, `perf` — as well as the [sysctl](Kernel-Tuning-with-sysctl.md) knobs used to codify the fixes. It complements [the broader hardening course](../Readme.md) by treating performance as a security-adjacent concern: resource exhaustion is a denial-of-service vector, and misconfigured limits (`ulimit`, cgroups, `/proc/sys` tunables) are as often abused by attackers as they are by noisy neighbors.

> [!NOTE]
> **Measure before you tune**
> Never change a kernel parameter or restart a service to "fix" performance without first establishing a baseline. Every note in this module pairs a diagnostic step with the specific metric it explains — tune from evidence, not intuition.

## Learning Objectives

- Apply the USE method to systematically triage CPU, memory, disk, and network bottlenecks on a live system
- Read and interpret `top`, `vmstat`, `mpstat`, `free`, `iostat`, `ss`, and `sar` output with confidence
- Distinguish load average from CPU utilization, and identify run-queue vs. I/O-wait saturation
- Diagnose memory pressure, swap thrashing, and the OOM killer's decision process
- Profile disk latency and throughput per device, and correlate it with filesystem/application symptoms
- Analyze network throughput, retransmits, and socket backlog exhaustion
- Collect long-term historical performance data with `sar`/`sysstat` for capacity planning and post-incident review
- Use `perf` for CPU profiling, flame graphs, and syscall/function-level hotspot analysis
- Persist safe, tested `sysctl` tunables across reboots without regressing security posture

## Topics Covered

| Note | What it covers |
|---|---|
| [Performance-Analysis-Overview](Performance-Analysis-Overview.md) | The USE methodology, baseline collection, and a triage decision tree tying symptoms to the right tool |
| [CPU-Performance-Analysis](CPU-Performance-Analysis.md) | Load average vs. utilization, run queues, context switches, `top`/`mpstat`/`vmstat -w`, steal time on VMs |
| [Memory-Performance-Analysis](Memory-Performance-Analysis.md) | `free`, page cache vs. available memory, swap behavior, `/proc/meminfo`, the OOM killer, `vmstat` memory columns |
| [Disk-IO-Performance](Disk-IO-Performance.md) | `iostat -x`, `%util` vs. `await`, IOPS vs. throughput, I/O schedulers, filesystem and mount tuning |
| [Network-Performance-Analysis](Network-Performance-Analysis.md) | `ss`, `ip -s link`, bandwidth vs. latency, TCP retransmits/window scaling, socket buffer and backlog tuning |
| [System-Statistics-with-sar](System-Statistics-with-sar.md) | Installing `sysstat`, `sar` subsystems (CPU/memory/disk/network), historical data collection and review |
| [perf-and-Profiling](perf-and-Profiling.md) | `perf top`/`perf stat`/`perf record`, flame graphs, tracepoints, and syscall-level profiling |
| [Kernel-Tuning-with-sysctl](Kernel-Tuning-with-sysctl.md) | `/proc/sys` layout, `sysctl -w` vs. persistent `/etc/sysctl.d/`, safe VM/network/file-descriptor tunables |

## Practical Labs

1. **Baseline and stress a system** — On a lab VM, capture a clean baseline with `vmstat 1 10`, `iostat -x 1 10`, and `sar -u 1 10`. Then run `stress-ng --cpu 4 --vm 2 --vm-bytes 1G --timeout 60s` and re-capture the same metrics; identify which resource saturated first and correlate with `dmesg` for any OOM events.
2. **Find a disk bottleneck** — Use `fio` to generate a mixed random-read/write workload against a scratch device, watch `iostat -x 1` in a second terminal, and identify the point where `%util` approaches 100% and `await` climbs disproportionately to `svctm`. Then compare `noop`/`mq-deadline` vs. `bfq` schedulers on the same device.
3. **Profile a CPU-bound process** — Write or run a simple CPU-intensive script, use `perf top -p <pid>` to find the hottest function, then `perf record -g -p <pid> -- sleep 10` followed by `perf report` to read the call graph. Cross-check against `pidstat -u 1` output for the same PID.

## Best Practices

- Always establish a baseline during normal operation before declaring anything "abnormal" — one-off snapshots without context are misleading.
- Prefer `sar`/`sysstat` for continuous historical data over ad-hoc one-time tool runs; you cannot retroactively diagnose an incident you didn't capture.
- Tune one variable at a time and re-measure — stacking multiple `sysctl` changes at once makes regressions impossible to attribute.
- Treat `/etc/sysctl.d/*.conf` as version-controlled configuration, not a scratchpad; document *why* each non-default value was set.
- Prefer cgroup-based resource limits (`systemd` slices, `cgroups v2`) over global `ulimit` changes when isolating a single service's resource ceiling.
- Validate tuning changes under representative load in a staging environment before promoting to production.

## Security Considerations

- Resource-exhaustion tuning is a DoS control surface: unbounded `fs.file-max`, `net.core.somaxconn`, or per-process `ulimit -u` values let a single compromised or buggy process starve the host — align limits with CIS Benchmark recommendations for the relevant RHEL/Debian release.
- `perf` requires elevated privileges (`CAP_PERFMON`/`CAP_SYS_ADMIN` or root) and can expose kernel addresses and other processes' memory contents; restrict access via `kernel.perf_event_paranoid` and only grant it to trusted administrators.
- Hardening controls that disable `/proc` visibility (e.g., `hidepid=2` on `/proc`) will break some diagnostic tools for non-root users — document this trade-off rather than silently loosening it to "make monitoring work."
- Historical `sar` data can leak sensitive workload/traffic patterns; restrict `/var/log/sa/` permissions and retention consistent with your data-handling policy.
- Never disable kernel security mitigations (e.g., Spectre/Meltdown patches via `mitigations=off`) purely for a performance win in production without an explicit, documented risk acceptance.

> [!NOTE]
> **📸 Screenshot**
> _Capture: `sar -u 1 5` (CPU), `free -h`, and `iostat -x 1 3` output side by side in a terminal, showing a system under moderate load with annotated columns of interest_

## Troubleshooting

| Symptom | Likely Cause | Where to Look |
|---|---|---|
| High load average, low CPU utilization | Processes blocked on I/O or uninterruptible sleep (`D` state) | [Disk-IO-Performance](Disk-IO-Performance.md), `ps aux \| awk '$8=="D"'` |
| System sluggish despite "free" memory looking fine | Page cache reclaim pressure or swap thrashing | [Memory-Performance-Analysis](Memory-Performance-Analysis.md), `vmstat 1`, `si`/`so` columns |
| Intermittent request latency spikes | Disk `await` spikes or network retransmits | [Disk-IO-Performance](Disk-IO-Performance.md), [Network-Performance-Analysis](Network-Performance-Analysis.md) |
| Process killed unexpectedly | OOM killer invoked under memory pressure | `dmesg \| grep -i "oom\|killed process"`, [Memory-Performance-Analysis](Memory-Performance-Analysis.md) |
| `sysctl` change lost after reboot | Applied with `sysctl -w` only, not persisted | [Kernel-Tuning-with-sysctl](Kernel-Tuning-with-sysctl.md), `/etc/sysctl.d/` |
| `perf` reports "Permission denied" | `kernel.perf_event_paranoid` too restrictive for non-root | [perf-and-Profiling](perf-and-Profiling.md), [Kernel-Tuning-with-sysctl](Kernel-Tuning-with-sysctl.md) |

## References

- Brendan Gregg — *Systems Performance: Enterprise and the Cloud* (USE Method, performance methodology)
- `man 1 top`, `man 1 vmstat`, `man 1 iostat`, `man 1 sar`, `man 1 perf`
- Red Hat Enterprise Linux Performance Tuning Guide — access.redhat.com
- `sysstat` project documentation — https://github.com/sysstat/sysstat
- CIS Benchmarks for RHEL/Ubuntu — kernel and resource-limit hardening sections

## Related Notes

- [Performance-Analysis-Overview](Performance-Analysis-Overview.md)
- [CPU-Performance-Analysis](CPU-Performance-Analysis.md)
- [Memory-Performance-Analysis](Memory-Performance-Analysis.md)
- [Disk-IO-Performance](Disk-IO-Performance.md)
- [Network-Performance-Analysis](Network-Performance-Analysis.md)
- [System-Statistics-with-sar](System-Statistics-with-sar.md)
- [perf-and-Profiling](perf-and-Profiling.md)
- [Kernel-Tuning-with-sysctl](Kernel-Tuning-with-sysctl.md)
- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
