# Multiple Commands and Pipes

## Overview

The Linux shell can do far more than run one command at a time. Using **control operators** (`;`, `&&`, `||`, `&`) you can chain commands into workflows, run conditionally on success or failure, and push work into the background. Using **pipes** (`|`) you can connect the output of one command to the input of another, building powerful data-processing chains from small single-purpose tools.

This "do one thing well, then compose" philosophy is central to UNIX. Mastering command chaining and piping turns the shell from a command runner into a lightweight automation and analysis environment.

## Concepts

| Operator | Name | Behaviour |
|----------|------|-----------|
| `;` | Semicolon | Run commands sequentially, regardless of success or failure |
| `&&` | Logical AND | Run the next command only if the previous one **succeeded** (exit 0) |
| `\|\|` | Logical OR | Run the next command only if the previous one **failed** (exit ≠ 0) |
| `&` | Ampersand | Run the command in the **background**, freeing the terminal |
| `\|` | Pipe | Send the **stdout** of one command to the **stdin** of the next |

```mermaid
flowchart LR
  A[command1] -->|exit code| D{status?}
  D -->|; always| B[command2]
  D -->|&& if success| C[command2]
  D -->|\|\| if failure| E[command2]
```

## Exit Codes

Every command returns a numeric **exit status** when it finishes: `0` means success, any non-zero value means some kind of failure. Control operators such as `&&` and `||` make their decisions based on this value.

### Zero Exit Codes (Success)

- The command below shows the **exit status** of the last executed command:

	- `$?` stores the exit status of the last command

	- `echo` prints that value

```bash
echo $?
```

> Example:

```bash
ls
```

```bash
echo $?
```

> Output: `0` → command executed successfully

### Non-zero Exit Codes (Failure)

- Output: non-zero value (commonly `2`)

```bash
ls nonexistentfile
```

```bash
echo $?
```

- Meaning:

| Code | Meaning |
|------|---------|
| 0 | Success |
| ≠ 0 | Failure |

## Multiple Commands

### Why Use Multiple Commands?

- Automate tasks

- Save time

- Execute workflows in one line

### 1. Using `;` (Semicolon)

- Executes commands sequentially

- Runs all commands regardless of success or failure

```bash
command1; command2; command3
```

> Example:

```bash
ifconfig; date; ls
```

### 2. Using `&&` (Logical AND)

- Executes next command only if previous succeeds (exit code `0`)

- Stops execution on failure

```bash
command1 && command2
```

> Example:

```bash
cd /etc && ls && echo "Listed /etc"
```

```bash
ifconfig && ls nonexistentfile && date
```

### 3. Using `||` (Logical OR)

- Executes next command only if previous fails

```bash
command1 || command2
```

> Example:

```bash
cd /not-exist || echo "Directory not found"
```

### 4. Using `&` (Background Execution)

- Runs command in the background

- Terminal remains usable

```bash
command &
```

> Example:

```bash
ping 8.8.8.8 > /dev/null &
```

- Check background jobs:

```bash
jobs
```

### Combined Example

The `&&` / `||` pair is a compact idiom for "if the command worked, do X, otherwise do Y".

```bash
ping -c 1 8.8.8.8 > /dev/null && echo "Host reachable" || echo "Host down"
```

```bash
ping -c 1 192.168.1.1 > /dev/null && echo "Host reachable" || echo "Host down"
```

> [!NOTE]
> The `cmd && A || B` idiom is convenient but has a subtle trap: if `A` itself fails, `B` still runs. For strict if/then/else logic in scripts, prefer an explicit `if … then … else … fi` block.

## Foreground and Background Processes (fg & bg)

### Foreground Processes

- A **foreground process** runs in the terminal and **occupies it**
- You cannot execute other commands until it finishes or is stopped

> Example

```bash
ping 8.8.8.8
```

- The terminal is busy until you stop it using:

    - `Ctrl + C` → terminate

    - `Ctrl + Z` → suspend

### Background Processes

- A **background process** runs **without blocking the terminal**

- Allows you to continue executing other commands

#### Run a Command in Background

```bash
command &
```

> Example

```bash
sleep 30 &
```

> Output:

```text
[1] 1234
```

- `[1]` → Job ID

- `1234` → Process ID (PID)

### Job Control

#### List Jobs

- Displays all background and suspended jobs

```bash
jobs
```

#### Suspend a Foreground Process

- Stops (pauses) the current foreground process

Press:

```text
Ctrl + Z
```

#### Resume in Background (`bg`)

- Resumes the most recently stopped job in the background

```bash
bg
```

- Resume Specific Job

```bash
bg %1
```

#### Bring to Foreground (`fg`)

- Brings the most recent background job to the foreground

```bash
fg
```

- Bring Specific Job

```bash
fg %1
```

### Example Workflow

```bash
ping 8.8.8.8
```

1. Press `Ctrl + Z` to suspend

2. Resume in background:

```bash
bg
```

3. Check jobs:

```bash
jobs
```

4. Bring back to foreground:

```bash
fg
```

### Useful Commands Summary

| Command | Description |
|---------|-------------|
| `&` | Run command in background |
| `jobs` | List jobs |
| `bg` | Resume stopped job in background |
| `fg` | Bring job to foreground |
| `Ctrl + Z` | Suspend current process |
| `Ctrl + C` | Terminate current process |

### Key Points

- Foreground processes block the terminal

- Background processes allow multitasking

- Use `Ctrl + Z` to pause a process

- Use `bg` and `fg` to manage jobs

- Use `jobs` to view active jobs

> [!TIP]
> A background job started with `&` is still tied to your shell and will receive a hangup (`SIGHUP`) if you log out. To survive logout, prefix with `nohup`, use `disown` after launching, or run the job inside a [screen](Screen-Command.md) session.

## Pipes (`|`)

### What are Pipes?

- Pipes (`|`) allow you to pass output of one command as input to another.

```bash
command1 | command2
```

### Common Examples

- View long output

```bash
ls -l /etc | less
```

- Search for text

```bash
cat /etc/passwd | grep root
```

- Sort output

```bash
ls /etc | sort
```

- Count lines

```bash
cat /etc/passwd | wc -l
```

- Top CPU-consuming processes

```bash
ps aux | sort -nrk 3 | head -n 10
```

### Important Note on Pipes

- Incorrect usage:

	- Pipes pass data, not execution flow

	- Commands like `pwd`, `date`, and `ls` do not use input from previous commands

```bash
ifconfig | pwd | date | ip ad | ls
```

> Correct usage:

```bash
cat /etc/passwd | grep root | sort | uniq
```

> [!WARNING]
> A pipe only connects **stdout** to the next command's **stdin**. Commands that ignore stdin (like `pwd`, `date`, `ls`) gain nothing from being on the right side of a pipe — the earlier output is silently discarded. Chain a pipe only when each stage actually consumes input.

## Pipes vs Redirection

| Feature | Pipe (`\|`) | Redirection (`>`) |
|---------|-------------|-------------------|
| Data Flow | Command → Command | Command → File |
| Storage | Temporary | Saved to file |
| Use Case | Processing output | Saving output |

Example:

```bash
ls | less        # pipe
```

```bash
ls > files.txt   # redirect
```

## Practical Examples

- Sequential execution

```bash
pwd; date; ls
```

- Conditional execution (success)

```bash
cd /var && ls && echo "Success"
```

- Conditional execution (failure)

```bash
cd /fake || echo "Failed"
```

- Background execution

```bash
sleep 10 & date
```

- Pipe with filtering

```bash
ps aux | grep apache
```

## Summary

- `;` → Run commands sequentially (always)

- `&&` → Run next command only if previous succeeds

- `||` → Run next command only if previous fails

- `&` → Run command in background

- `|` → Pass output from one command to another

## References

- `man 1 bash` — *Lists & Pipelines* and *Job Control* sections cover `;`, `&&`, `||`, `&`, and `|`.
- GNU Bash Reference Manual — [Pipelines](https://www.gnu.org/software/bash/manual/html_node/Pipelines.html) and [Job Control](https://www.gnu.org/software/bash/manual/html_node/Job-Control.html).
- Advanced Bash-Scripting Guide — [Exit and Exit Status](https://tldp.org/LDP/abs/html/exit-status.html).

## Related
- [Standard-Data-Streams](Standard-Data-Streams.md) — stdin/stdout/stderr piping works on
- [Cat-Command](Cat-Command.md) — common source of piped input
- [grep-Command](../String-Processing-and-Finding-Files/grep-Command.md) — filter piped output
- [Screen-Command](Screen-Command.md) — keep background jobs alive after logout
- Remote-Code-Execution-to-Reverse-shell — redirection builds reverse shells
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
