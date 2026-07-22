# Comparison Operators

## Overview

Comparison operators are the tests that drive conditional logic in Bash. Inside `[ ... ]` (the POSIX `test` builtin) or `[[ ... ]]` (Bash's extended test), an operator compares two values and resolves to a true/false result expressed as an [exit status](Exit-status.md) — `0` for true, non-zero for false. Those results feed [if statements](IF-Statement.md), `while` loops, and `&&`/`||` chains.

Bash groups its operators by the kind of data they compare:

- **String comparison** — equality, inequality, ordering, regex match.
- **String tests** — is a string empty or non-empty.
- **Integer comparison** — numeric equality and ordering.
- **File tests** — does a path exist, and what type/permissions does it have.
- **File comparison** — compare two paths by age or inode.

> [!IMPORTANT]
> Use **numeric** operators (`-eq`, `-lt`, …) for integers and **string** operators (`==`, `!=`, `<`, `>`) for text. Comparing `"10" -gt "9"` is true (numeric), but `"10" > "9"` is **false** (lexicographic, "1" sorts before "9"). Mixing them is a classic scripting bug.

## Concepts

### String comparison

| Operator | Purpose |
|----------|---------|
| `==` (also `=`) | String equality |
| `!=` | String inequality |
| `<` | String lexicographic comparison (before) |
| `>` | String lexicographic comparison (after) |
| `=~` | String regular expression match |

> [!WARNING]
> `<`, `>`, and `=~` require the `[[ ... ]]` construct. Inside single-bracket `[ ... ]`, an unescaped `<`/`>` is interpreted as I/O redirection, silently corrupting the test (and possibly creating a file). Prefer `[[ ... ]]` for all string comparisons.

### String tests

| Operator | Purpose |
|----------|---------|
| `-z "string"` | String has zero length |
| `-n "string"` | String has non-zero length |

### Integer comparison

| Operator | Purpose |
|----------|---------|
| `-eq` | Integer equality |
| `-ne` | Integer inequality |
| `-lt` | Integer less than |
| `-le` | Integer less than or equal to |
| `-gt` | Integer greater than |
| `-ge` | Integer greater than or equal to |

### File tests

| Operator | Purpose |
|----------|---------|
| `-a file` | Exists (use `-e` instead) |
| `-b file` | Exists and is a block special file |
| `-c file` | Exists and is a character special file |
| `-d file` | Exists and is a directory |
| `-e file` | Exists |
| `-f file` | Exists and is a regular file |
| `-g file` | Exists and is set-group-id |
| `-h file` | Exists and is a symbolic link |
| `-k file` | Exists and its sticky bit is set |
| `-p file` | Exists and is a named pipe (FIFO) |
| `-r file` | Exists and is readable |
| `-s file` | Exists and has a size greater than zero |
| `-t fd` | Descriptor `fd` is open and refers to a terminal |
| `-u file` | Exists and its set-user-id bit is set |
| `-w file` | Exists and is writable |
| `-x file` | Exists and is executable |
| `-O file` | Exists and is owned by the effective user id |
| `-G file` | Exists and is owned by the effective group id |
| `-L file` | Exists and is a symbolic link |
| `-S file` | Exists and is a socket |
| `-N file` | Exists and has been modified since it was last read |

### File comparison

| Operator | Purpose |
|----------|---------|
| `file1 -nt file2` | `file1` is newer (according to modification date) than `file2`, or `file1` exists and `file2` does not |
| `file1 -ot file2` | `file1` is older than `file2`, or `file2` exists and `file1` does not |
| `file1 -ef file2` | `file1` is a hard link to `file2` |

## Examples

### Numeric comparison in a condition

```bash
#!/bin/bash
if [ "$#" -lt 2 ]; then
  echo "Usage: $0 <a> <b>"
  exit 1
fi
```

### String emptiness test

```bash
#!/bin/bash
read -p "Enter username: " USER
if [ -z "$USER" ]; then
  echo "No username supplied"
  exit 1
fi
```

### File-type and permission checks

```bash
#!/bin/bash
FILE="/etc/shadow"
if [ -f "$FILE" ] && [ -r "$FILE" ]; then
  echo "$FILE is a readable regular file"
fi
```

### Regex match with `[[ ... ]]`

```bash
#!/bin/bash
IP="192.168.1.10"
if [[ "$IP" =~ ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
  echo "Looks like an IPv4 address"
fi
```

## Best Practices

> [!TIP]
> - Always **quote** both operands: `[ "$a" = "$b" ]`. An unquoted empty variable turns `[ $a = $b ]` into a syntax error or a wrong result.
> - Use `[[ ... ]]` in Bash for string and regex work; reserve `[ ... ]` for POSIX-portable scripts.
> - Use `(( ... ))` for pure arithmetic comparisons: `if (( a < b ))` is clearer than `[ "$a" -lt "$b" ]`.

## Security Considerations

- Permission tests (`-r`, `-w`, `-x`, `-u`, `-O`) are the backbone of privilege-escalation and hardening checks — e.g. flagging world-writable files or unexpected SUID bits (`-u`). Combine `-u`/`-g` with `find` audits when reviewing a host against CIS Benchmarks.
- File tests are subject to **TOCTOU** (time-of-check/time-of-use) races: a path validated with `-f`/`-w` can be swapped for a symlink before you act on it. For security-sensitive operations, operate on file descriptors or use atomic primitives rather than test-then-use.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `[: too many arguments` | Unquoted variable expanded to multiple words | Quote operands: `[ "$var" = x ]` |
| `[: unary operator expected` | Variable is empty/undefined | Quote it and/or provide a default: `[ "${var:-}" = x ]` |
| String `<`/`>` gives odd results or creates a file | Used inside `[ ... ]` where `<`/`>` is redirection | Switch to `[[ ... ]]` |
| Numeric compare always false | Used string operator on numbers | Use `-eq`/`-lt`/… or `(( ... ))` |

## Related

- [Shell-Scripting](Shell-Scripting.md) — parent guide for shell language constructs
- [IF-Statement](IF-Statement.md) — comparison operators drive conditional tests
- [Exit-status](Exit-status.md) — comparisons return a true/false exit code
- [Variables](Variables.md) — values being compared are held in variables
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
