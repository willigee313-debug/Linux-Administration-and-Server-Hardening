# Shells and Environment

Bash, Zsh, and Fish; environment variables, startup files, and prompt customization.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

The shell is the administrator's primary interface. This module compares Bash, Zsh, and Fish, explains how environment variables and startup files shape a session, and shows how to customize prompts and tooling (including grc for colorized output) for a productive, consistent working environment.

## Learning Objectives

By the end of this module you will be able to:

- Distinguish login vs non-login and interactive vs non-interactive shells and their startup files
- Set and export environment variables persistently and per-session
- Customize the shell and prompt and choose an appropriate shell per use case

## Topics Covered

This module contains **5 notes**.

| Note | Topic |
| --- | --- |
| [Enhance-Bash-with-grc](Enhance-Bash-with-grc.md) | Enhance Bash with grc |
| [Fish-Shell](Fish-Shell.md) | Fish Shell |
| [Linux-Environment-Variables](Linux-Environment-Variables.md) | Linux Environment Variables |
| [Shells-in-Linux](Shells-in-Linux.md) | Shells in Linux |
| [Zsh-Shell](Zsh-Shell.md) | Zsh Shell |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Keep interactive customizations in the right rc file so scripts stay fast and predictable
- Use `/etc/profile.d/*.sh` for system-wide environment changes
- Pin the default login shell for service accounts to `nologin` where interactive access is unwanted

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Never place secrets in world-readable shell rc files or in `PATH`-searched directories
- Avoid a writable `.` in `PATH` to prevent trojaned-binary execution

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Environment variable set but not visible in a program | It was set in an rc file not read for that shell type; export it in the correct startup file |
| PATH changes lost after reboot | Persist them in a login startup file, not just the current session |

## References

- [GNU Bash manual](https://www.gnu.org/software/bash/manual/)
- [Zsh documentation](https://zsh.sourceforge.io/Doc/)
- [Fish shell docs](https://fishshell.com/docs/current/)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Shell Scripting](../Shell-Scripting/Readme.md) — related module
- [Linux Basic Commands](../Linux-Basic-Commands/Readme.md) — related module
- [Text Editors](../Text-Editors/Readme.md) — related module
