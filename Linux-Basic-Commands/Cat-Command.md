# Cat Command

## Overview

The `cat` command (short for **concatenate**) is one of the most frequently used text utilities on Linux and other UNIX-like systems. Despite its simple purpose — writing the contents of files to standard output — it is a versatile building block for viewing files, joining them together, creating short files, appending data, and feeding text into pipelines and redirections.

Because `cat` reads from standard input when no file is given and writes to standard output by default, it composes cleanly with pipes (`|`) and redirection operators (`>`, `>>`), making it a natural companion to almost every other command-line tool.

> [!TIP]
> For simply *reading* a large file, prefer a pager such as `less`. Reserve `cat` for concatenation, quick views of small files, and stream processing where its output is piped or redirected.

## Concepts

| Term | Meaning |
|------|---------|
| Concatenate | Join two or more files end to end into a single output stream |
| Standard output (stdout) | Default destination `cat` writes to — the terminal, unless redirected |
| Standard input (stdin) | Source `cat` reads from when no filename is supplied |
| Redirection | Sending output to a file with `>` (overwrite) or `>>` (append) |
| Heredoc | Inline block of text (`<< EOF … EOF`) fed to a command as input |

```mermaid
flowchart LR
  F1[file1.txt] --> C{{cat}}
  F2[file2.txt] --> C
  STDIN[keyboard / stdin] --> C
  C --> OUT[stdout / terminal]
  C --> RED[> or >> file]
  C --> PIPE[| next command]
```

## Basic Syntax

```bash
cat [OPTION] [FILE]
```

## Commands

### Display File Contents

- Displays the contents of a file to the terminal.

```bash
cat filename.txt
```

```bash
cat /etc/passwd
```

> Example Output

```text
root:x:0:0:root:/root:/bin/bash
user:x:1000:1000:user:/home/user:/bin/bash
```

### Display Multiple Files

- Displays multiple files sequentially.

```bash
cat file1.txt file2.txt
```

```bash
cat /etc/passwd /etc/group
```

- Using Separate Commands

```bash
cat /etc/passwd; cat /etc/group
```

### Combine Files Into a New File

- Concatenates multiple files and saves the output into another file.

```bash
cat file1.txt file2.txt > combined.txt
```

```bash
cat test.txt text1.txt > newfile.txt
```

```bash
cat /etc/hostname /etc/hosts > system-info.txt
```

- Combine All Matching Files

```bash
cat /etc/*.conf > all-conf.txt
```

### Append Content to Existing File

> Appends content instead of overwriting.

- Interactive Append

```bash
cat >> test.txt
```

> Type content and press:

```text
Ctrl + D
```

to save and exit.

- Append Another File

```bash
cat /etc/hosts >> test.txt
```

```bash
cat list.txt >> t1.txt
```

### Create a New File

- Creates a file interactively.

```bash
cat > test.txt
```

> Type content:

```text
Hello
Linux
```

> Press:

```text
Ctrl + D
```

> to save.

### Create File Using Here Document (Heredoc)

- Useful for creating multi-line files inside scripts.

```bash
cat << EOF > notes.txt
Line 1
Line 2
Line 3
EOF
```

> Example

```bash
cat << EOF > users.txt
admin
user1
user2
EOF
```

## Formatting Options

### Display Line Numbers

- Number All Lines

```bash
cat -n test.txt
```

- Example Output

```text
     1  Hello
     2  Linux
     3  Testing
```

### Show Special Characters

- Show End of Line (`$`)

```bash
cat -e test.txt
```

> Example Output

```text
Hello$
Linux$
```

- Show Line Numbers and End Characters

```bash
cat -ne test.txt
```

### Suppress Repeated Blank Lines

- The `-s` option removes consecutive empty lines.

```bash
cat -s filename.txt
```

> Before

```text
Line1


Line2
```

> After

```text
Line1

Line2
```

### View Large Files with Paging

- Using `more`

```bash
cat test.txt | more
```

- Using `less`

```bash
cat test.txt | less
```

- Recommended Alternative, Directly use:

```bash
less test.txt
```

> [!TIP]
> Piping `cat` into a pager (`cat file | less`) is a common but unnecessary pattern — it spawns an extra process for no benefit. Open the file with the pager directly: `less file`.

### Commonly Used Options

| Option | Description |
|--------|-------------|
| `-n` | Show line numbers |
| `-e` | Show `$` at end of each line |
| `-s` | Suppress repeated blank lines |
| `-T` | Display TAB characters as `^I` |
| `-b` | Number non-empty lines |
| `-A` | Show all special characters |

## Examples

- Copy File Contents

```bash
cat source.txt > destination.txt
```

- Merge Log Files

```bash
cat access.log error.log > combined.log
```

- Display Hidden Characters

```bash
cat -A test.txt
```

- Create Multi-Line Configuration

```bash
cat << EOF > app.conf
PORT=8080
DEBUG=true
HOST=0.0.0.0
EOF
```

### Difference Between `>` and `>>`

| Operator | Function |
|----------|----------|
| `>` | Overwrites file |
| `>>` | Appends to file |

> Example

- Overwrites `test.txt`.

```bash
cat file1.txt > test.txt
```

- Appends to `test.txt`.

```bash
cat file1.txt >> test.txt
```

> [!WARNING]
> `>` truncates the target file to zero length **before** the command runs. A mistyped `cat a > a` will destroy the file's contents. Double-check the destination before redirecting.

## Best Practices

- Use `less` (not `cat`) to browse large or unknown-length files; `cat` on a multi-gigabyte log floods the terminal and wastes memory in the scrollback.
- Prefer `>>` when adding to logs or existing config so you do not accidentally overwrite prior content.
- In scripts, use heredocs (`<< EOF … EOF`) to generate configuration files reproducibly instead of chained `echo` statements.
- Avoid the "useless use of cat" anti-pattern: `cat file | grep pattern` can usually be written `grep pattern file`.

## Security Considerations

- Printing files such as `/etc/passwd` is harmless (it contains no password hashes), but `cat`-ing sensitive files like `/etc/shadow`, private keys, or credential files echoes secrets into your terminal scrollback and shell history context — be deliberate about where that output lands.
- Redirecting with `>` as root can silently overwrite critical system files. Confirm the destination path, and consider `set -o noclobber` in interactive shells to make `>` refuse to overwrite existing files.
- When building config files via heredoc in scripts, ensure the resulting file permissions are appropriate (e.g. `chmod 600`) if it contains secrets.

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---------|--------------|------------|
| `cat: file: No such file or directory` | Wrong path or typo in filename | Verify with `ls`; check the working directory with `pwd` |
| `cat: file: Permission denied` | File not readable by current user | Check permissions with `ls -l`; use `sudo` if appropriate |
| Terminal fills with garbage / bell | `cat` on a binary file | Use `file` to check type; use `xxd` or `hexdump` for binary |
| `cat > file` appears to hang | Waiting for input on stdin | Type content, then press `Ctrl + D` (EOF) to finish |

## References

- `man 1 cat` — GNU coreutils manual page for `cat`.
- GNU Coreutils Manual — [Output of entire files: `cat`](https://www.gnu.org/software/coreutils/manual/html_node/cat-invocation.html).
- POSIX.1-2017 — `cat` utility specification (The Open Group Base Specifications).

## Related
- [Multiple-Commands-and-Pipes](Multiple-Commands-and-Pipes.md) — pipe cat output into other tools
- [Standard-Data-Streams](Standard-Data-Streams.md) — stdin/stdout/stderr that cat reads and writes
- [grep-Command](../String-Processing-and-Finding-Files/grep-Command.md) — filter the lines cat prints
- [Nano-Command](../Text-Editors/Nano-Command.md) — edit the files you view with cat
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
