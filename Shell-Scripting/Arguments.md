# Arguments

## Overview

Arguments (also called **positional parameters**) are values passed into a Bash script or function at invocation time. They let a single script behave differently depending on the input it receives, which is the foundation of reusable, non-interactive automation — exactly what you want for cron jobs, CI pipelines, and hardening scripts that must run without a human at the keyboard.

You can pass arguments in two contexts:

- **Into a script** — values supplied on the command line after the script name.
- **Into a function** — values supplied after the function name when it is called (see [Functions](Functions.md)).

> [!NOTE]
> Example of calling a script with three arguments:
> `./arguments one two three`

A Bash script exposes these parameters through a set of special variables. The basic positional parameters run from `$1` to `$9`; anything beyond nine must be brace-wrapped (`${10}`, `${11}`, …).

## Concepts

### Special shell parameters

| Parameter | Meaning |
|-----------|---------|
| `$0` | Name of the script (the command as invoked) |
| `$1` … `$9` | Positional parameters for arguments one to nine |
| `${10}` … `${n}` | Positional parameters for arguments after nine (braces required) |
| `$*` | All the arguments as a single string |
| `$@` | Same as `$*`, but differs when enclosed in `"` (double quotes) |
| `$#` | Total number of arguments |
| `$$` | PID of the running script |
| `$?` | Last return code (exit status of the previous command) |

> [!IMPORTANT]
> The distinction between `"$*"` and `"$@"` matters for correctness. When double-quoted, `"$*"` expands to a **single** word (all arguments joined by the first character of `IFS`), while `"$@"` expands to **separate** words — one per argument, preserving whitespace and empty arguments. Always iterate with `"$@"` unless you specifically want one joined string.

### Internal Field Separator (IFS)

`IFS` controls how the shell splits strings into words. Its default value is space, tab, and newline. You can inspect it with:

```bash
# set | grep IFS
	IFS=$' \t\n\C-@'
```

Overriding `IFS` changes how `$*` joins arguments and how word-splitting behaves throughout the script.

## Examples

### Reading positional parameters (`arguments.sh`)

```bash
#!/bin/bash
echo "Script name: $0"
echo "Frist argument: $1"
echo "Second argument: $2"
echo "Third argument: $3"
echo "10th argument: ${10}"
echo "All argument with \$*: $*"
echo "All argument with \$@: $@"
echo "Argument count \$#: $#"
```

Invoke it with more than nine arguments to see why `${10}` needs braces:

```bash
# ./arguments.sh one two three 4 five 6 7 8 9 10 11
```

### Changing the field separator (`arguments2.sh`)

Setting `IFS=","` changes how `$*` joins the argument list into a single string.

```bash
chmo#!/bin/bash
IFS=","
echo "Script name: $0"
echo "Frist argument: $1"
echo "Second argument: $2"
echo "Third argument: $3"
echo "10th argument: ${10}"
echo "All argument with \$*: $*"
echo "All argument with \$@: $@"
echo "Argument count \$#: $#"
```

### Arithmetic on numeric arguments (`arguments3.sh`)

```bash
#!/bin/bash

FIRST=$1

SECOND=$2

let RESULT=FIRST+SECOND

echo "$FIRST + $SECOND = $RESULT"
```

Run it with two numbers:

```bash
# ./arguments.sh 4 5
```

## Best Practices

> [!TIP]
> - **Always quote expansions**: use `"$1"`, `"$@"`, and `"${10}"` to survive filenames and values containing spaces.
> - **Validate argument count** with `$#` before using parameters: `if [ "$#" -lt 2 ]; then echo "Usage: $0 <a> <b>"; exit 1; fi`.
> - **Prefer named options** (`getopts`) over long positional chains once a script takes more than two or three inputs — see [getopts](getopts.md).
> - **Never `eval` raw arguments** and never interpolate them unquoted into commands; that is a classic shell-injection path.

## Security Considerations

Arguments are untrusted input. Under CIS/NIST hardening guidance, treat every parameter as attacker-controlled:

- Validate and constrain values (e.g. confirm a path is inside an expected directory) before acting on them.
- Avoid passing secrets as command-line arguments — they are visible in the process table (`ps -ef`) to any local user. Prefer environment variables or files with restrictive permissions.
- Quote consistently to prevent word-splitting and glob expansion turning `$1` into unintended commands or paths.

## Related

- [Shell-Scripting](Shell-Scripting.md) — parent guide these positional parameters belong to
- [Variables](Variables.md) — arguments are accessed as special shell variables
- [getopts](getopts.md) — parsing named option arguments
- [Functions](Functions.md) — functions take their own positional arguments
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
