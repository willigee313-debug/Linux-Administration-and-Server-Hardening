# Shell Scripting

Bash scripting fundamentals: variables, arguments, conditionals, functions, and getopts.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Automation begins with shell scripting. This module builds from variables and positional arguments through comparison operators, if-statements, arrays, functions, exit status, and getopts-based option parsing — the constructs needed to write robust, reusable administration scripts with proper error handling.

## Learning Objectives

By the end of this module you will be able to:

- Write parameterized scripts using positional arguments and getopts
- Use conditionals, comparison operators, and arrays to express logic
- Structure code with functions and propagate meaningful exit status

## Topics Covered

This module contains **9 notes**.

| Note | Topic |
| --- | --- |
| [Arguments](Arguments.md) | Arguments |
| [Arrays](Arrays.md) | Arrays |
| [Comparison-Operators](Comparison-Operators.md) | Comparison Operators |
| [Exit-status](Exit-status.md) | Exit status |
| [Functions](Functions.md) | Functions |
| [IF-Statement](IF-Statement.md) | IF Statement |
| [Shell-Scripting](Shell-Scripting.md) | Shell Scripting |
| [Variables](Variables.md) | Variables |
| [getopts](getopts.md) | getopts |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Start scripts with `set -euo pipefail` for fail-fast, predictable behavior
- Quote all variable expansions (`"$var"`) to avoid word-splitting bugs
- Validate inputs and return non-zero on error so callers can react

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Never build shell commands from unvalidated input; avoid `eval`
- Create temporary files with `mktemp` and restrictive umask
- Do not embed credentials in scripts — read them from protected files or a secrets store

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Script works interactively but fails in cron | cron has a minimal environment; set explicit PATH and use absolute paths |
| `[: too many arguments` | An unquoted empty variable broke the test; quote expansions |

## References

- [GNU Bash manual](https://www.gnu.org/software/bash/manual/)
- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
- [ShellCheck](https://www.shellcheck.net/)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Automation](../Automation/Readme.md) — scaling scripts into Ansible and scheduled jobs
- [Shells and Environment](../Shells-and-Environment/Readme.md) — related module
- [String Processing and Finding Files](../String-Processing-and-Finding-Files/Readme.md) — related module
- [Process, Service and Job Management](../Process-Service-and-Job-Management/Readme.md) — related module
