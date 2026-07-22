# String Processing and Finding Files

Text processing with grep, sed, awk, cut, paste; locating files with find, locate, which.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

The classic UNIX text-processing toolkit and file-location commands. This is one of the largest modules in the course, covering regular expressions and grep, stream editing with sed, field-oriented processing with awk, column tools like cut and paste, and the family of file-finding utilities (find, locate, which, whereis, type, tree). Mastery here powers log analysis, automation, and one-liners used everywhere else.

## Learning Objectives

By the end of this module you will be able to:

- Search and filter text with grep and regular expressions
- Transform streams non-interactively with sed (substitute, delete, insert, append)
- Process delimited data with awk fields, records, separators, and BEGIN/END blocks
- Locate files and binaries efficiently with find, locate, which, and whereis

## Topics Covered

This module contains **31 notes**.

| Note | Topic |
| --- | --- |
| [$1-$2-Dollars-everywhere]($1-$2-Dollars-everywhere.md) | $1 $2 Dollars everywhere |
| [-e-option-Run-multiple-sed-commands](-e-option-Run-multiple-sed-commands.md) | e option Run multiple sed commands |
| [-i-option-Changing-files-for-sure](-i-option-Changing-files-for-sure.md) | i option Changing files for sure |
| [Arithmetic](Arithmetic.md) | Arithmetic |
| [Field-Separator](Field-Separator.md) | Field Separator |
| [File-Comparison-Tools-in-Linux](File-Comparison-Tools-in-Linux.md) | File Comparison Tools in Linux |
| [File-Finding-in-Linux](File-Finding-in-Linux.md) | File Finding in Linux |
| [Find-Command](Find-Command.md) | Find Command |
| [Number-of-Fields](Number-of-Fields.md) | Number of Fields |
| [Number-of-Records](Number-of-Records.md) | Number of Records |
| [Record-Separator](Record-Separator.md) | Record Separator |
| [Searching-Pattern](Searching-Pattern.md) | Searching Pattern |
| [String-Processing](String-Processing.md) | String Processing |
| [a-(append)-and-i(prepand)](a-(append)-and-i(prepand).md) | a (append) and i(prepand) |
| [awk-Command](awk-Command.md) | awk Command |
| [c-(change-command)](c-(change-command).md) | c (change command) |
| [cut-Command](cut-Command.md) | cut Command |
| [d-(delete-command)](d-(delete-command).md) | d (delete command) |
| [e-(perform-shell-commands)](e-(perform-shell-commands).md) | e (perform shell commands) |
| [grep-Command](grep-Command.md) | grep Command |
| [locate](locate.md) | locate |
| [p-(print-command)-and--n-option](p-(print-command)-and--n-option.md) | p (print command) and n option |
| [paste-Command](paste-Command.md) | paste Command |
| [print-BEGIN{}-{}-END{}](print-BEGIN{}-{}-END{}.md) | print BEGIN{} {} END{} |
| [q-(quit-command)](q-(quit-command).md) | q (quit command) |
| [s-(substitute-command)](s-(substitute-command).md) | s (substitute command) |
| [sed](sed.md) | sed |
| [tree](tree.md) | tree |
| [type](type.md) | type |
| [whereis](whereis.md) | whereis |
| [which](which.md) | which |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Prefer `find -print0 | xargs -0` for filenames with spaces or newlines
- Test `sed`/`awk` transforms on a copy before using `-i` in place
- Anchor regular expressions to avoid over-matching

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Be cautious with `sed -i` and `find -exec rm` — they modify or delete irreversibly
- Validate untrusted input before feeding it to `awk`/`eval`-style constructs

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| `grep` returns nothing for a pattern that clearly exists | Special regex characters may need escaping, or use `grep -F` for fixed strings |
| `sed -i` changed the wrong lines | Restore from backup; add an address range or anchor to scope the edit |

## References

- [GNU grep manual](https://www.gnu.org/software/grep/manual/)
- [GNU sed manual](https://www.gnu.org/software/sed/manual/)
- [GNU Awk User's Guide](https://www.gnu.org/software/gawk/manual/)
- [find(1) man page](https://man7.org/linux/man-pages/man1/find.1.html)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Linux Basic Commands](../Linux-Basic-Commands/Readme.md) — related module
- [Shell Scripting](../Shell-Scripting/Readme.md) — related module
- [Shells and Environment](../Shells-and-Environment/Readme.md) — related module
