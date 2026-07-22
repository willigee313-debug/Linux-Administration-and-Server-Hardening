# getopts

## Overview

Parsing command-line options by hand quickly becomes unwieldy: you must handle short flags (`-h`), long flags (`--help`), options that take values, options that are optional, and arbitrary ordering. The `getopt` utility solves this by providing a defined syntax to declare and parse arguments, normalising whatever the user typed into a clean, predictable stream your script can loop over.

Reference: <https://stackabuse.com/how-to-parse-command-line-arguments-in-bash/>

```bash
# type -a getopts
```

```bash
# type -a getopt
```

> [!NOTE]
> There are two related tools. `getopts` is a **Bash builtin** that handles short options only. `getopt` is an **external GNU utility** (`/usr/bin/getopt`) that also handles long options via `--longoptions`. This note uses the external `getopt`, which is why long flags like `--alpha` work.

## Concepts

### Short vs long arguments

There are two kinds of arguments when passing them to a command-line utility:

| Kind | Form | Example |
| --- | --- | --- |
| Short arguments | Single character prefixed by one hyphen | `-h` (help), `-l` (list) |
| Long arguments | Whole word prefixed by two hyphens | `--help`, `--list` |

Short arguments are passed to the `--options` flag of the `getopt` utility; long arguments are passed to the `--longoptions` flag.

### Declaring whether an option takes a value

The colons after an option letter/name control whether it consumes a value:

| Notation | Meaning |
| --- | --- |
| No colon | No value is required (a plain flag, e.g. `-h`) |
| Single colon `:` | A value is **required** for this option |
| Double colon `::` | A value is **optional** |

> [!NOTE]
> The help option in command-line utilities doesn't take any values, and hence it doesn't have a colon attached to it.

### Parsing pipeline

```mermaid
flowchart TD
    A["$@ (raw user arguments)"] --> B["getopt --options SHORT --longoptions LONG"]
    B --> C["Normalised string ending in --"]
    C --> D['eval set -- "$OPTS"']
    D --> E[while : / case $1 loop]
    E --> F{Match option}
    F -->|"-a | --alpha"| G["ALPHA=$2; shift 2"]
    F -->|"--"| H[shift; break]
    F -->|"*"| I[Unexpected option]
    G --> E
```

## Configuration

Two variables define the accepted option set and are reused throughout the examples:

```text
SHORT=l:,o:,h            # short options: -l VALUE, -o VALUE, -h (flag)
LONG=list:,output:,help  # long options:  --list VALUE, --output VALUE, --help (flag)
```

Key `getopt` flags:

| Flag | Purpose |
| --- | --- |
| `--options` / `-o` | Comma/letter list of short options |
| `--longoptions` | Comma list of long options |
| `--name` / `-n` | Program name used in error messages |
| `--alternative` / `-a` | Allow long options to start with a single `-` |
| `--quiet` / `-q` | Suppress `getopt`'s own error output |

## Examples

### getopts-1.sh — declare and echo the parsed options

```bash
#!/bin/bash
SCRIPT_NAME=$0
SHORT=l:,o:,h
LONG=list:,output:,help
OPTS=$(getopt --alternative --name $SCRIPT_NAME --options $SHORT --longoptions $LONG -- "$@")
echo $OPTS
```

Invoke it with a mix of short and long options:

```bash
$ ./getopts-1.sh -l ip.txt -o /tmp -h --list --output --help
```

### getopts-2.sh — short flag equivalents (`-a`, `-n`)

```bash
#!/bin/bash

SHORT=l:,o:,h
LONG=list:,output:,help
OPTS=$(getopt -a -n $SCRIPT_NAME --options $SHORT --longoptions $LONG -- "$@")
echo $OPTS
```

> [!NOTE]
> `getopt` appends a trailing double hyphen (`--`) to mark the end of the options. Everything after it is treated as positional arguments, which a loop can then iterate over. This is where the `shift` keyword helps: `shift` takes an optional count of how many positions to advance the argument cursor, so `shift 2` consumes both an option and its value before moving on.

### getopts-3.sh — normalising with `eval set`

Reading the standardised argument list back into the shell's positional parameters makes the script behave as if it were called with the simpler, normalised set.

```bash
#!/bin/bash

SHORT=l:,o:,h
LONG=list:,output:,help
OPTS=$(getopt -a -n $SCRIPT_NAME --options $SHORT --longoptions $LONG -- "$@")
echo "OPTS before eval : $OPTS"
eval set -- "$OPTS"
echo "OPTS after eval  : $OPTS"
```

### getopts-4.sh — the `eval set` idiom

> [!IMPORTANT]
> `eval` is necessary when arguments contain spaces, and you must quote the variable too — i.e. `eval set -- "$args"`. By reading that set of standardised arguments into the shell's input arguments, the script now thinks it was called with the simpler, standardised set.

```bash
eval set -- "$OPTS"
```

> [!WARNING]
> `eval` re-parses and executes its argument. It is safe **only** because the string comes from `getopt`, which quotes each token. Never pass unsanitised user input directly to `eval` — doing so is a direct command-injection vector.

### final-example.sh — full option parser

A complete parser: default values, a usage/help function, `getopt` validation via `$?`, and a `while` + `case` loop that assigns each option's value with `shift 2`.

```bash
#!/bin/bash
SCRIPT_NAME=$0

# Set some default values:
ALPHA=unset
BRAVO=unset
CHARLIE=unset
DELTA=unset

USAGE_HELP
function USAGE_HELP()
{
  echo "Usage: $SCRIPT_NAME [ -a | --alpha var]
                      [ -b | --bravo var]
                      [ -c | --charlie var]
                      [ -d | --delta   var]"
  exit 2
}


SHORT=a:,b:,c:,d:,h
LONG=alpha:,bravo:,charlie:,delta:,help
PARSED_ARGUMENTS=$(getopt --alternative --quiet --name $SCRIPT_NAME --options $SHORT --longoptions $LONG -- "$@")
VALID_ARGUMENTS=$?
if [ "$VALID_ARGUMENTS" != "0" ]; then
  USAGE_HELP
fi


eval set -- "$PARSED_ARGUMENTS"
unset PARSED_ARGUMENTS

while :
do
  case "$1" in

    -h | --help)

      help
      shift ;;

    -a | --alpha)

      ALPHA=$2
      echo "ALPHA : $ALPHA"
      shift 2 ;;

    -b | --bravo)

      BRAVO=$2
      echo "BRAVO : $BRAVO"
      shift 2 ;;

    -c | --charlie)

      CHARLIE=$2
      echo "CHARLIE : $CHARLIE"
      shift 2 ;;

    -d | --delta)

      DELTA=$2
      echo "DELTA : $DELTA"
      shift 2 ;;

    # -- means the end of the arguments; drop this, and break out of the while loop
    --) shift; break ;;

    # If invalid options were passed, then getopt should have reported an error,
    # which we checked as VALID_ARGUMENTS when getopt was called...

    *)

      echo "Unexpected option: $1"
      help ;;

  esac
done
```

> [!WARNING]
> In the snippet above `USAGE_HELP` is called on the line before its `function USAGE_HELP()` definition. A function must be defined before it is invoked, so define `USAGE_HELP()` first — as the corrected variant below does.

### Corrected variant — define before use, require at least one argument

This version defines `USAGE_HELP()` before calling it, and also shows usage when no arguments are supplied (`[ "$#" == "0" ]`). Option patterns are single-quoted.

```bash
#!/bin/bash
SCRIPT_NAME=$0

# Set some default values:
ALPHA=unset
BRAVO=unset
CHARLIE=unset
DELTA=unset

function USAGE_HELP()
{
  echo "Usage: $SCRIPT_NAME [ -a | --alpha var]
                      [ -b | --bravo var]
                      [ -c | --charlie var]
                      [ -d | --delta   var]"
  exit 2
}


SHORT=a:,b:,c:,d:,h
LONG=alpha:,bravo:,charlie:,delta:,help
PARSED_ARGUMENTS=$(getopt --alternative --quiet --name $SCRIPT_NAME --options $SHORT --longoptions $LONG -- "$@")
VALID_ARGUMENTS=$?
if [ "$VALID_ARGUMENTS" != "0" ] || [ "$#" == "0" ]; then
  USAGE_HELP
fi


eval set -- "$PARSED_ARGUMENTS"
unset PARSED_ARGUMENTS

while :
do
  case "$1" in

    '-h' | '--help')

      USAGE_HELP
      shift ;;

    '-a' | '--alpha')

      ALPHA=$2
      echo "ALPHA : $ALPHA"
      shift 2 ;;

    '-b' | '--bravo')

      BRAVO=$2
      echo "BRAVO : $BRAVO"
      shift 2 ;;

    '-c' | '--charlie')

      CHARLIE=$2
      echo "CHARLIE : $CHARLIE"
      shift 2 ;;

    '-d' | '--delta')

      DELTA=$2
      echo "DELTA : $DELTA"
      shift 2 ;;

    '--')

      shift
      break ;;

    '*')

      help ;;

  esac
done
```

## Best Practices

- Always quote `"$@"` when handing arguments to `getopt`, and quote the result before `eval set -- "$OPTS"`.
- Check `getopt`'s exit status (`$?`) immediately and print usage on failure.
- Set sensible defaults for every option variable before the parse loop.
- Define helper functions (usage/help) **before** they are called.
- Use `shift 2` for value-taking options and `shift` (or `shift 1`) for flags so the cursor stays aligned.
- Prefer the external `getopt` for long-option support; use the `getopts` builtin for short-only, maximally portable scripts.

## Security Considerations

> [!WARNING]
> The `eval set -- "$OPTS"` idiom is the security-sensitive heart of this pattern. It is safe only because `getopt` emits properly quoted tokens. Do **not** feed raw, unquoted, or user-assembled strings to `eval`. Validate option values (paths, hostnames, numbers) against an allowlist before using them in privileged commands, and reject unexpected input in the `*)` catch-all case.

## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `USAGE_HELP: command not found` | Function called before it is defined | Move the `function USAGE_HELP()` block above its first call |
| Long options rejected | Using the `getopts` builtin instead of `getopt` | Use external `/usr/bin/getopt` with `--longoptions` |
| Values with spaces split apart | Missing quotes around `eval set` | Use `eval set -- "$OPTS"` with quotes |
| Infinite loop | Missing `shift` in a `case` branch | Ensure every branch shifts and `--` breaks the loop |

## References

- <https://stackabuse.com/how-to-parse-command-line-arguments-in-bash/> — How to parse command-line arguments in Bash
- `man getopt` (external util) and `help getopts` (Bash builtin)
- GNU `getopt(1)` — util-linux documentation

## Related

- [Shell-Scripting](Shell-Scripting.md) — parent guide for shell language constructs
- [Arguments](Arguments.md) — getopt parses positional/option arguments
- [Functions](Functions.md) — wrapping option parsing in a function
- [IF-Statement](IF-Statement.md) — branching on parsed options
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
