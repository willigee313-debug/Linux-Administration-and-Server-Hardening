# Automation

## Overview

Automation turns repetitive, error-prone Linux administration into repeatable, auditable, version-controlled work. This module covers [Ansible](Ansible-Introduction.md) for agentless configuration management and orchestration at fleet scale, [Bash scripting](Bash-Automation.md) for local task automation, and the two native Linux job schedulers — [cron](Cron-Automation.md) and [systemd timers](systemd-Timers.md) — for running work on a schedule. Together these tools let an administrator manage tens or thousands of hosts with the same rigor as application code: peer-reviewed, tested, and rolled back when something breaks.

> [!TIP]
> **Why this module matters**
> Manual, one-off changes made by SSHing into a box and running commands are the single biggest cause of configuration drift and "it worked yesterday" incidents. Every topic in this module exists to make changes **declarative, idempotent, and reproducible** instead of tribal knowledge trapped in someone's shell history.

## Learning Objectives

- Explain the agentless push model Ansible uses and why it needs only SSH and Python on managed nodes
- Write and run Ansible playbooks that define desired system state idempotently
- Structure reusable automation with Ansible roles instead of monolithic playbooks
- Encrypt secrets at rest in version control using Ansible Vault
- Write robust, defensive Bash scripts (error handling, logging, argument parsing) for local automation
- Schedule recurring jobs with cron, including user crontabs, `/etc/cron.d`, and `run-parts` directories
- Schedule and monitor recurring jobs with systemd timers as a modern, loggable alternative to cron
- Choose the right automation tool (Bash script vs. cron vs. systemd timer vs. Ansible) for a given task

## Topics Covered

| Note | What it covers |
|---|---|
| [Ansible-Introduction](Ansible-Introduction.md) | Ansible architecture, inventory, ad-hoc commands, modules, and the control node/managed node model |
| [Ansible-Playbooks](Ansible-Playbooks.md) | YAML playbook syntax, tasks, handlers, variables, idempotency, and `ansible-playbook` execution |
| [Ansible-Roles](Ansible-Roles.md) | Role directory structure, reusability, `ansible-galaxy`, and composing playbooks from roles |
| [Ansible-Vault](Ansible-Vault.md) | Encrypting secrets/variables at rest, `ansible-vault` commands, and vault password management |
| [Bash-Automation](Bash-Automation.md) | Defensive shell scripting patterns: `set -euo pipefail`, functions, logging, argument parsing, exit codes |
| [Cron-Automation](Cron-Automation.md) | crontab syntax, user vs. system crontabs, `/etc/cron.d`, `anacron`, and logging cron output |
| [systemd-Timers](systemd-Timers.md) | `.timer`/`.service` unit pairs, `OnCalendar` syntax, `systemctl list-timers`, and journal-based logging |

## Practical Labs

1. **Idempotent web server rollout** — Write an Ansible playbook that installs and configures Apache/Nginx on 2-3 target hosts from an inventory file, run it twice, and verify the second run reports zero changes (proving idempotency).
2. **Vault-protected role** — Build an Ansible role that deploys a service requiring a database password; store the password with `ansible-vault encrypt_string` and confirm the playbook fails safely without the vault password supplied.
3. **Backup job, two ways** — Implement the same nightly backup task (a Bash script that tars `/etc` and rotates old archives) first as a cron job in `/etc/cron.d`, then as a systemd timer/service pair; compare failure visibility using `journalctl -u backup.service` vs. cron's mail/log output.

## Best Practices

- Keep automation code (playbooks, roles, scripts, unit files) in version control (Git) with meaningful commit messages — treat infrastructure code like application code.
- Prefer Ansible modules over `shell`/`command` tasks whenever a module exists; modules are idempotent and report accurate change state, raw shell invocations are not.
- Always run `ansible-playbook --check --diff` (dry run) before applying changes to production inventory.
- Write Bash scripts with `set -euo pipefail`, quote all variable expansions, and validate inputs before acting on them.
- Prefer systemd timers over cron for new work on systemd-managed distros — you get dependency ordering, resource limits (via the paired service's `[Service]` directives), and structured logging for free.
- Use `run-parts`-style drop-in directories (`/etc/cron.d/`, `/etc/cron.daily/`) instead of editing a monolithic crontab, so packages and configuration management can safely add/remove jobs.
- Name and document every scheduled job (cron comment or systemd unit `Description=`) so on-call staff can identify what a job does without reading its source.

## Security Considerations

- **Least privilege**: run Ansible tasks and cron/systemd jobs as an unprivileged service account wherever possible; reserve `become`/root execution for tasks that genuinely require it (CIS control: minimize privileged automation).
- **Secrets management**: never commit plaintext passwords, API keys, or SSH private keys to a playbook or script — use Ansible Vault, environment files with restrictive permissions (`chmod 600`), or an external secrets manager.
- **SSH hygiene**: Ansible's control node needs SSH key access to every managed node — protect that private key like a root credential; consider a dedicated, IP-restricted automation user with `AllowUsers`/`sudo` NOPASSWD scoped to specific commands only.
- **Script permissions**: automation scripts invoked by root-owned cron jobs or systemd services must not be group/world-writable (`chmod 750` or stricter); a writable script run as root is a local privilege-escalation path.
- **Audit trail**: prefer systemd timers or Ansible's built-in logging (`ansible.cfg` `log_path`) over silent cron jobs so every automated change is attributable and reviewable — align with NIST 800-53 AU-2 (auditable events).
- **Vault password storage**: don't hardcode the Ansible Vault password in a script or repo; use `--vault-password-file` pointing at a file outside version control, or integrate with a secrets manager via a vault password script.

> [!NOTE]
> **📸 Screenshot**
> _Capture: `systemctl list-timers --all` output alongside `crontab -l` for the same host, showing both scheduling mechanisms side by side_

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Ansible playbook hangs on "Gathering Facts" | SSH connectivity or `python3` missing on managed node | Verify `ansible all -m ping`; install `python3` on target |
| Cron job never runs | Wrong crontab syntax, missing newline at EOF, or `PATH` not set | Validate with `crontab -l`, ensure trailing newline, set `PATH=` explicitly in crontab |
| systemd timer shows `inactive` and never fires | Timer enabled but not started, or `OnCalendar` syntax error | `systemctl enable --now foo.timer`; test expression with `systemd-analyze calendar "<expr>"` |
| Bash script fails silently in cron but works interactively | Missing environment variables (cron runs a minimal shell) | Source `/etc/profile` or set required vars explicitly at the top of the script |
| Ansible Vault-encrypted var fails at runtime | Vault password file not passed or wrong password | Re-run with `--ask-vault-pass` or `--vault-password-file`; confirm password matches |

## References

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Vault Documentation](https://docs.ansible.com/ansible/latest/vault_guide/index.html)
- `man crontab`, `man 5 crontab`
- [systemd.timer(5) man page](https://www.freedesktop.org/software/systemd/man/systemd.timer.html)
- [Bash Pitfalls (Greg's Wiki)](https://mywiki.wooledge.org/BashPitfalls)

## Related Notes

- [Ansible-Introduction](Ansible-Introduction.md) — Ansible architecture and core concepts
- [Ansible-Playbooks](Ansible-Playbooks.md) — writing and running playbooks
- [Ansible-Roles](Ansible-Roles.md) — structuring reusable automation
- [Ansible-Vault](Ansible-Vault.md) — encrypting secrets in automation code
- [Bash-Automation](Bash-Automation.md) — defensive shell scripting for local tasks
- [Cron-Automation](Cron-Automation.md) — traditional job scheduling with cron
- [systemd-Timers](systemd-Timers.md) — modern job scheduling with systemd
- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
