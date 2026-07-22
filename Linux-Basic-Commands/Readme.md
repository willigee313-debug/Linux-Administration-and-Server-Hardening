# Linux Basic Commands

Core command-line building blocks: navigation, viewing, streams, pipes, and archiving.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

A practical tour of the everyday commands every administrator relies on. It covers reading and concatenating files, chaining commands with pipes, the three standard data streams, terminal multiplexing with screen, and compressing and archiving data. These primitives underpin every later module, from scripting to service configuration.

## Learning Objectives

By the end of this module you will be able to:

- Navigate and inspect the filesystem confidently from the shell
- Combine commands using pipes and redirect the three standard streams
- Create and extract archives with tar/gzip and manage detached sessions with screen

## Topics Covered

This module contains **6 notes**.

| Note | Topic |
| --- | --- |
| [Cat-Command](Cat-Command.md) | Cat Command |
| [Compress-and-Archive](Compress-and-Archive.md) | Compress and Archive |
| [Linux-Basic-Commands](Linux-Basic-Commands.md) | Linux Basic Commands |
| [Multiple-Commands-and-Pipes](Multiple-Commands-and-Pipes.md) | Multiple Commands and Pipes |
| [Screen-Command](Screen-Command.md) | Screen Command |
| [Standard-Data-Streams](Standard-Data-Streams.md) | Standard Data Streams |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Prefer piping and composition over one-off manual steps
- Use `screen`/`tmux` for long-running jobs over SSH so a dropped connection does not kill work
- Quote and escape file arguments to survive spaces and special characters

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Be deliberate with redirection (`>`) — it truncates target files without warning
- Avoid leaking secrets into shell history when piping credentials

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Command output is empty but exit status is 0 | A pipe may be swallowing stderr; separate `2>&1` handling |
| `screen` session lost after logout | Reattach with `screen -r`; use `-d -r` to detach elsewhere first |

## References

- [GNU Coreutils manual](https://www.gnu.org/software/coreutils/manual/)
- [The Linux Documentation Project](https://tldp.org/)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [String Processing and Finding Files](../String-Processing-and-Finding-Files/Readme.md) — related module
- [Shells and Environment](../Shells-and-Environment/Readme.md) — related module
- [Shell Scripting](../Shell-Scripting/Readme.md) — related module
