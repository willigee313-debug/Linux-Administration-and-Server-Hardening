# Flashcards — CLI, Text Processing & Files

Spaced-repetition cards covering standard I/O streams, command chaining, `cat`/archiving, the `grep`/`sed`/`awk`/`cut` text-processing toolkit, `find`/`locate`/`which`/`type`/`whereis`, Vim/Nano editing, and shells/environment variables — drawn from the Linux Basic Commands, String Processing and Finding Files, Text Editors, and Shells and Environment modules.

## Streams, Redirection & Job Control

What file descriptor number is stdin?::0
What file descriptor number is stdout?::1
What file descriptor number is stderr?::2
Which redirection operator truncates/overwrites a file, and which appends?::`>` overwrites (truncates first); `>>` appends
Why does `2>&1` need to come *after* `> file` to merge stderr into the file?::`2>&1` points stderr at wherever stdout currently points — if written before `> file`, stderr still goes to the terminal
Which Bash shell option makes `>` refuse to overwrite an existing file?::`set -o noclobber`
In `cmd1 && cmd2`, when does `cmd2` run?::Only if `cmd1` exits with status 0 (success)
In `cmd1 || cmd2`, when does `cmd2` run?::Only if `cmd1` fails (non-zero exit status)
What key suspends a foreground process, and what command resumes it in the background?::`Ctrl+Z` suspends it; `bg` resumes it in the background
What signal does a background job (`&`) receive when the launching shell exits or logs out?::SIGHUP

## cat, Archiving & screen

Which `cat` option suppresses repeated blank lines?::`-s`
What is the "useless use of cat" anti-pattern, and its fix?::`cat file | grep pattern` — pipe a file into grep instead of just running `grep pattern file` directly
Which tar flags together create a gzip-compressed archive with verbose output?::`-czvf`
Which tar flag is used to decompress a bzip2-compressed archive on extraction?::`-j` (e.g. `tar -xjvf archive.tbz`)
Do `tar`, `gzip`, `bzip2`, and `xz` support built-in password protection?::No — only `zip` (`-e`) and `7z` (`-p`) support password protection natively
Which command starts a named, detached screen session running a command in one line?::`screen -S <name> -dm <command>`
What key sequence detaches from a running screen session without killing it?::`Ctrl+a` then `d`

## grep, sed & awk

What does `grep -E` enable that basic `grep` does not?::Extended regular expressions (`+`, `?`, `|`, `()` without escaping)
Which grep option prints only the matched text instead of the whole line?::`-o`
Which sed command substitutes a pattern on every occurrence in a line (not just the first)?::`s/old/new/g`
Which sed option edits a file in place while keeping a `.bak` backup copy?::`-i.bak`
In an awk program, which block runs exactly once, before any input is read?::`BEGIN{}`
Which awk flag sets the field separator used to split each record?::`-F` (e.g. `awk -F: '{print $1}'`)

## cut, find & locate

Which cut option selects fields, and which option specifies the delimiter?::`-f` selects fields; `-d` sets the delimiter
What is the key operational difference between `find` and `locate`?::`find` walks the live filesystem in real time; `locate` queries a prebuilt index database (faster, but can be stale)
Which command rebuilds/refreshes the database that `locate` searches?::`updatedb`
What find expression hunts for SUID binaries system-wide (a classic privesc audit)?::`find / -type f -perm -04000` (or `-perm -04000 -ls`)
What does `find / -perm -o=w -type f` search for?::World-writable files
How does `find ... -exec cmd {} +` differ from `... -exec cmd {} \;`?::`+` batches many matched files into fewer command invocations (faster); `\;` runs the command once per file

## which, whereis & type

Which command reports only the on-disk executable path resolved from `$PATH`, ignoring shell aliases and functions?::`which`
Which command reflects exactly what the shell will run for a name — including aliases, functions, and builtins?::`type` (a shell builtin)
What does `whereis` report that plain `which` does not?::A command's man page and source file locations, in addition to the binary

## Vim & Nano

What keystroke sequence starts recording a Vim macro into register `a`, and what stops it?::`qa` starts recording; `q` (while recording) stops it
How do you replay Vim macro register `a` ten times?::`10@a`
What is the Vim `:substitute` command syntax?::`:[range]s/pattern/replacement/[flags]`
Which Vim substitute flag prompts for y/n confirmation before each replacement?::`c`
Which Vim range in a substitute command targets the entire file?::`%`
What should be the first line of a hand-written `~/.vimrc`, and why?::`set nocompatible` — without it Vim may fall back to strict vi behavior and disable multi-level undo and other modern features
Which Vim option disables execution of potentially dangerous in-file modelines?::`set nomodeline`
In Nano, which keystroke saves ("writes out") the current file?::`Ctrl+O`
In Nano, which keystroke exits the editor?::`Ctrl+X`
Which tool should you use instead of a plain editor to safely edit `/etc/sudoers`?::`visudo` (validates syntax before saving)

## Shells & Environment Variables

What is the key difference between a shell variable and an environment variable?::A shell variable exists only in the current shell (`var=value`); an environment variable is exported and inherited by child processes (`export var=value`)
Which command lists only exported environment variables, as opposed to `set` which lists all shell variables and functions?::`env` (or `printenv`)
Why must `export` NOT be used inside `/etc/environment`?::That file is read by PAM (`pam_env`), not by a shell, so it only accepts plain `KEY=value` lines
Why is a `.` (current directory) entry in `$PATH` a security risk?::It lets command resolution fall back to the current directory, allowing an attacker's planted binary (e.g. `./ls`) to run instead of the trusted system tool

## Related
- [Linux Basic Commands](../Linux-Basic-Commands/Readme.md)
- [String Processing and Finding Files](../String-Processing-and-Finding-Files/Readme.md)
- [Text Editors](../Text-Editors/Readme.md)
- [Shells and Environment](../Shells-and-Environment/Readme.md)
- [Exam Preparation](Readme.md)
- [Linux Administration & Server Hardening](../Readme.md)
