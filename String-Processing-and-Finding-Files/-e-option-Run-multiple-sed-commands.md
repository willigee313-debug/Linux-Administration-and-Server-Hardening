# `-e` Option — Run Multiple `sed` Commands

## Overview

The `-e` (expression) option lets a single `sed` invocation carry **multiple editing commands**. Each `-e 'command'` adds one expression to the script, and `sed` applies them to every line in the order they were written. This is how you chain print, substitute, append, insert, delete, and quit operations without piping through several `sed` processes.

> [!NOTE]
> `-e` is the explicit way to supply a command. A lone `sed 'command' file` implies a single `-e`. You only *need* `-e` when supplying two or more commands (or when a command starts with a character that would otherwise be misread).

## Concepts

| Option | Meaning | Typical use |
|---|---|---|
| `-e 'cmd'` | Add one command to the sed script | Chain several edits in one pass |
| `-n` | Suppress automatic printing | Print only what a `p` command emits |
| `p` | Print the current pattern space | Show matching lines |
| `a text` | Append `text` after the line | Add lines below matches |
| `i text` | Insert `text` before the line | Add lines above matches |
| `q[code]` | Quit (optionally with exit code) | Stop early after a match |

General syntax:

```bash
sed -e 'command1' -e 'command2' file
```

```mermaid
flowchart LR
    L["Line from file"] --> C1["-e command1"]
    C1 --> C2["-e command2"]
    C2 --> C3["-e command3"]
    C3 --> O["Output (auto-print unless -n)"]
```

## Commands

### Print multiple patterns

- Print lines matching `armour` and `root` using `-n` to suppress automatic output.

```bash
sed -ne '/armour/p' -ne '/root/p' user-list.txt
```

### Multiple print commands without `-n`

- Same as above, but automatic printing remains enabled.

```bash
sed -e '/armour/p' -e '/root/p' user-list.txt
```

> [!WARNING]
> Without `-n`, every matching line appears **twice** — once from the explicit `p` command and once from `sed`'s automatic print. Add `-n` when you want each match printed exactly once.

### Combine print and quit

- Print matching `armour` lines, then quit with exit code `2`.

```bash
sed -ne '/armour/p' -ne '/armour/q2' user-list.txt
```

### Print matching lines from `/etc/passwd`

- Print lines matching `armour`.

```bash
sed -ne '/armour/p' /etc/passwd
```

### Print and quit in `/etc/passwd`

- Print matching lines and quit with exit code `2`.

```bash
sed -ne '/armour/p' -ne '/armour/q2' /etc/passwd
```

### Check exit status

- Display the exit code from the previous `sed` command.

```bash
echo $?
```

> [!TIP]
> A custom quit code (`q2`) lets a shell script branch on whether a pattern was found. Read it immediately with `echo $?` before running any other command overwrites it.

### Combine append and insert

- Append `Armour User` and insert separator lines around matches of `armour`.

```bash
sed -e '/armour/a Armour User' -e '/armour/i "-----------"' user-list.txt
```

### Insert plain separator lines

- Same operation using plain dashes.

```bash
sed -e '/armour/a Armour User' -e '/armour/i -----------' user-list.txt
```

### Append and insert in `/etc/passwd`

- Add custom text around matching `armour` entries.

```bash
sed -e '/armour/a Armour User' -e '/armour/i armour user' /etc/passwd
```

> [!WARNING]
> These commands print the modified stream to **stdout** — the real `/etc/passwd` is not changed. To write changes back to a file you must add `-i` (see [-i-option-Changing-files-for-sure](-i-option-Changing-files-for-sure.md)). Never edit `/etc/passwd` in place without a backup.

### Prefix every line

- Prefix every line from `in.txt` with `prefix`.

```bash
cat in.txt | sed -e "s/.*/prefix&/" > out.txt
```

> [!NOTE]
> `&` in the replacement stands for the whole matched text, so `s/.*/prefix&/` re-emits each line with `prefix` prepended.

## Best Practices

- Pair `-n` with explicit `p` commands to control output precisely and avoid duplicate lines.
- Keep each logical edit in its own `-e` expression — it reads more clearly and is easier to reorder or comment out.
- Remember that commands run **in written order**; place filters/quits where they should take effect.
- Test the pipeline without `-i` first; only add in-place editing once the output is exactly right.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Lines printed twice | Used `p` without `-n` | Add `-n` |
| `sed` reads a command as a filename | Missing `-e` before a second command | Prefix each command with `-e` |
| Exit code always `0` | Read `$?` after another command ran | Capture `echo $?` right after the `sed` call |
| File unchanged | `-e` prints to stdout only | Add `-i` (with a `.bak` backup) |

## References

- GNU sed Manual: <https://www.gnu.org/software/sed/manual/sed.html>
- POSIX `sed` specification: <https://pubs.opengroup.org/onlinepubs/9699919799/utilities/sed.html>

## Related
- [sed](sed.md) — parent sed command
- [s-(substitute-command)](s-(substitute-command).md) — common command chained with -e
- [d-(delete-command)](d-(delete-command).md) — another command stacked via -e
- [p-(print-command)-and--n-option](p-(print-command)-and--n-option.md) — print command often combined with -n
- [-i-option-Changing-files-for-sure](-i-option-Changing-files-for-sure.md) — write chained edits back to disk
- [String-Processing](String-Processing.md) — text-processing hub
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
