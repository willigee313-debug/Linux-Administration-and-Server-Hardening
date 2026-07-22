# Variables

## Overview

A variable is a symbolic name for a chunk of memory to which we can assign values, read, and manipulate its contents. In shell scripting, variables carry configuration, user input, command results, and intermediate state through a script. Bash variables are untyped by default — everything is a string — but they can be treated as integers in arithmetic contexts.

> [!NOTE]
> There is **no space** around the `=` in an assignment. `VAR=value` is correct; `VAR = value` is parsed as running the command `VAR` with arguments `=` and `value`.

## Concepts

### Ways to assign a value

There are three ways to assign a value to a variable.

| Method | Syntax | Purpose |
| --- | --- | --- |
| Explicit definition | `VAR=value` | Assign a literal value directly |
| Read command | `read VAR` | Populate from user input (stdin) |
| Command substitution | `VAR=$(command)` | Capture a command's output |

**Explicit definition** examples:

```text
COUNT=5
PATH=/var/lib
NUMBER=8
MY_MESSAGE="Hello World"
```

> [!WARNING]
> `PATH` is a critical environment variable that tells the shell where to find executables. Overwriting it (as in `PATH=/var/lib` above, shown purely as a syntax example) will break command resolution for the rest of that shell. Use a different, non-reserved name for your own data.

**Read command:** `read VAR` — waits for a line of input and stores it in `VAR`.

**Command substitution:** `VAR=$(pwd)` — runs `pwd` and stores its output. The older backtick form `` VAR=`pwd` `` is equivalent but not nestable and harder to read.

### Accessing a variable

Prefix the name with `$` to expand (read) its value.

```bash
echo $COUNT
echo $PATH
echo "path = $PATH"
```

> [!TIP]
> Always wrap expansions in double quotes — `echo "$PATH"` — so values containing spaces or globs are preserved intact. Use `${VAR}` braces when the name is followed by other characters, e.g. `${USER_NAME}_file`.

## Examples

### var.sh — explicit assignment

```bash
#!/bin/bash
VAR=value
echo $VAR
MY_MESSAGE="Hello World"
echo $MY_MESSAGE
```

### var2.sh — reading input

```bash
#!/bin/bash
echo What is your name?
read MY_NAME
echo "Hello $MY_NAME - hope you're well."
```

### var3.sh — prompt on the same line with `echo -n`

```bash
#!/bin/bash
echo -n What is your name?:
read MY_NAME
echo "Hello $MY_NAME - hope you're well."
```

### var4.sh — prompts and hidden input with `read -p` / `read -sp`

`-p` supplies an inline prompt; `-s` suppresses echoing, which is essential for password entry.

```bash
#!/bin/bash
read -p "What is your name?: " MY_NAME
echo "Hello $MY_NAME - hope you're well."
echo
read -sp "What is your Password?: " MY_PASS
echo "Hello $MY_PASS - hope you're well."
```

> [!WARNING]
> The example echoes the password back only to demonstrate that `read -s` captured it. In real scripts **never print or log secrets**. Capturing input with `-s` prevents shoulder-surfing and keeps the value out of the terminal scrollback.

### var5.sh — reading from a file

Redirecting a file into `read` assigns its first line to the variable.

```bash
#!/bin/bash
read HOSTNAME < /etc/hostname
echo $HOSTNAME
```

### var6.sh — command substitution with `$( )`

```bash
#!/bin/bash
CURRENT_DIRECTORY=$(pwd)
echo $CURRENT_DIRECTORY
```

### var7.sh — command substitution with backticks

```bash
#!/bin/bash
CURRENT_DIRECTORY=`pwd`
echo "Your Current Directory is = $CURRENT_DIRECTORY"
```

### elapsed-time.sh — timing with `date +%s`

Capture epoch seconds before and after work, then subtract with arithmetic expansion `$(( ))`.

```bash
#!/bin/bash
START=$(date +%s)
echo $START
CURRENT_DIRECTORY=`pwd`
echo "Your Current Directory is = $CURRENT_DIRECTORY"
sleep 2
END=$(date +%s)
DIFFERENCE=$(( END - START ))
echo "Elapsed Time: $DIFFERENCE seconds."
```

### elapsed-time2.sh — the built-in `SECONDS` variable

Bash increments the special `SECONDS` variable automatically once it is set, giving elapsed time without manual arithmetic on timestamps.

```bash
#!/bin/bash
SECONDS=0
CURRENT_DIRECTORY=`pwd`
echo "Your Current Directory is = $CURRENT_DIRECTORY"
sleep 2
DURATION=$SECONDS
echo "Elapsed Time: $(($DURATION / 60)) minutes and $(($DURATION % 60)) seconds."
```

### var8.sh — braces in variable expansion

`${USER_NAME}_file` uses braces so the shell knows where the variable name ends and the literal suffix begins.

```bash
#!/bin/sh
echo "What is your name?"
read USER_NAME
echo "Hello $USER_NAME"
echo "I will create you a file called ${USER_NAME}_file"
touch "${USER_NAME}_file"
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: Terminal running var8.sh: prompts for a name, then confirms creation of a file named <name>_file_

## Best Practices

- Quote every expansion: `"$VAR"` — prevents word-splitting and glob expansion.
- Use `${VAR}` braces whenever the name abuts other text.
- Avoid clobbering reserved variables (`PATH`, `HOME`, `IFS`, `SECONDS`); choose descriptive uppercase names for your own globals and lowercase for locals.
- Prefer `$( )` over backticks — it nests and reads more clearly.
- Declare integers with `declare -i` (or use `$(( ))`) when doing arithmetic to avoid string surprises.

## Security Considerations

> [!WARNING]
> Variables are the primary vehicle for injection in shell scripts. An unquoted `$VAR` that contains attacker-controlled data can word-split into extra arguments or expand globs. `eval "$VAR"` and unquoted expansion in commands are classic remote-code-execution vectors.

- Never store passwords or tokens in plain variables that get exported to child processes or logged. Read secrets with `read -s` and unset them (`unset MY_PASS`) as soon as they are used.
- Sanitise and validate any variable sourced from user input, files, or the network before using it in a path, command, or test.
- Be mindful that environment variables are inherited by child processes and visible via `/proc/<pid>/environ` — do not pass secrets this way.

## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `VAR: command not found` | Spaces around `=` in assignment | Write `VAR=value` with no spaces |
| Value truncated at first space | Unquoted expansion word-split | Quote it: `echo "$VAR"` |
| `bad substitution` | POSIX `sh` used with a Bash-only construct | Use `#!/bin/bash` sha-bang |
| Empty output from substitution | Command wrote to stderr, not stdout | Redirect: `VAR=$(cmd 2>&1)` if appropriate |

## References

- GNU Bash Reference Manual — Shell Parameters & Parameter Expansion
- `man bash` (`Parameters`, `Command Substitution`)
- ShellCheck — <https://www.shellcheck.net/>

## Related

- [Shell-Scripting](Shell-Scripting.md) — parent guide for shell language constructs
- [Arguments](Arguments.md) — positional parameters are special variables
- [Arrays](Arrays.md) — multi-value variable type
- [Functions](Functions.md) — variable scope within functions
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
