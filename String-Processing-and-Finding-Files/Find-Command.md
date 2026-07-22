# find Command

## Overview

`find` walks a directory tree in real time and matches files against a rich set of tests — name, type, owner, permissions, size, and timestamps — then optionally runs an action on each match. Unlike `locate`, it queries the **live filesystem**, so results are always current at the cost of speed.

This note is a compact, copy-paste-friendly reference for everyday `find` usage, security auditing (SUID/SGID, world-writable, ownerless files), and safe bulk actions with `-exec` and `-delete`.

> [!TIP]
> **Mental model**
> `find <where> <what to match> <what to do>` — a starting path, one or more test expressions, and an optional action such as `-print` (default), `-exec`, or `-delete`.

## Concepts

| Component | Role | Examples |
|---|---|---|
| Starting point | Where the walk begins | `/`, `.`, `/var/log` |
| Tests | Filter matches by attribute | `-name`, `-type`, `-user`, `-perm`, `-mtime`, `-size` |
| Actions | What to do with matches | `-print`, `-exec`, `-delete`, `-ls` |
| Operators | Combine/negate tests | `-o` (OR), `!` (NOT), `\( \)` (grouping) |

## Install `findutils`

`find` is part of the `findutils` package, present by default on virtually all distributions. If it is missing:

- RHEL / CentOS / Fedora

```bash
yum install findutils
```

- Debian / Ubuntu

```bash
apt install findutils
```

## Basic Syntax

```bash
find [starting-point] [expression]
```

> Example:

```bash
find / -name passwd
```

## Find by Name

- Finds files by name

```bash
find . -name X.log
```

- Finds a file named `passwd` from current directory

```bash
find -name passwd
```

- Searches `/` for a file named `X.log`

```bash
find / -name X.log
```

- Searches for a file named `passwd` from `/`

```bash
find / -name passwd
```

- Searches current directory (`.`)

```bash
find .
```

## Wildcards & Pattern Matching

- Uses wildcards (glob) with escaped characters

```bash
find /root/ -name \*\.txt
```

- Finds files starting with `pass`

```bash
find -name pass*
```

```bash
find / -name pass*
```

- Case-insensitive search for `passwd`

```bash
find -iname passwd
```

- Case-insensitive wildcard match for `Pass*`

```bash
find / -iname Pass*
```

- Searches for a directory named `root`

```bash
find / -name root
```

> [!WARNING]
> **Quote your globs**
> When a pattern contains `*`, quote or escape it (`"*.txt"` or `\*\.txt`) so the **shell** does not expand it before `find` sees it.

## find -type: File Type Options

The `-type` option filters results by file type.

### Syntax

```bash
find [path] -type [c]
```

### File Type Codes

|Code|File Type|Description|
|---|---|---|
|`b`|Block special file|Buffered I/O device|
|`c`|Character special file|Unbuffered I/O device|
|`d`|Directory|Directories only|
|`p`|Named pipe (FIFO)|IPC file|
|`f`|Regular file|Standard file|
|`l`|Symbolic link|Symlink|
|`s`|Socket|IPC/network socket|
|`D`|Door (Solaris only)|Solaris IPC|

### Symbolic Links

- Finds symbolic links

```bash
find / -type l
```

- Finds symlinks even when using `-L`

```bash
find -L / -xtype l
```

### File & Directory Searches

- Finds a directory named `root`

```bash
find / -name root -type d
```

- Finds a file named `root`

```bash
find / -name root -type f
```

- Suppresses permission denied errors

```bash
find / -name root -type f 2> /dev/null
```

### User-Based Search

- Finds files owned by user `daemon`

```bash
find ./Documents/ -user daemon
```

- Finds files owned by `armour`

```bash
find / -user armour
```

```bash
find / -user armour -type f
```

```bash
find / -user armour -type d
```

- Finds `.bash*` files owned by `armour`

```bash
find / -user armour -name .bash\* 2> /dev/null
```

```bash
find / -user armour -name .bash\* -type f 2> /dev/null
```

### Empty Files & Directories

- Finds empty items in a directory

```bash
find ./Documents/ -empty
```

```bash
find ./Documents/ -empty -type f
```

```bash
find ./Documents/ -empty -type d
```

- System-wide empty file/dir search

```bash
find / -empty
```

```bash
find / -empty -type d
```

```bash
find / -empty -type f
```

```bash
find /var -empty
```

## Permissions & Ownership

### Readable / Writable / Executable

- Finds readable files

```bash
find / -readable -type f 2> /dev/null
```

- Finds writable directories

```bash
find / -writable -type d 2> /dev/null
```

- Finds executable files

```bash
find / -executable -type f 2> /dev/null
```

### Permission Searches

- Find files with permission `777`

```bash
find / -perm 777
```

```bash
find / -perm 777 -type f
```

```bash
find / -perm 777 -type d
```

- Find sticky-bit directories (`1777`)

```bash
find / -perm 1777 -type d 2> /dev/null
```

- Search for files with specific user/group IDs

```bash
find / -uid 1000
```

```bash
find / -gid 1000
```

```bash
find / -uid 1000 2> /dev/null
```

### Special Permission Bits

- World-writable files

```bash
find / -perm -o=w -type f 2>/dev/null
```

- Files with user `rwx`

```bash
find / -perm -u=rwx -type f 2>/dev/null
```

- Files with SGID bit

```bash
find / -perm -g=s -type f 2>/dev/null
```

- Files with SUID bit

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

> [!IMPORTANT]
> **Security auditing**
> SUID/SGID and world-writable searches are core hardening checks. A leading `-` in `-perm -04000` means "at least these bits set", which is exactly what you want when hunting privilege-escalation candidates. See [Setuid(Set-User-ID)](../Users-Groups-and-Permissions/Setuid(Set-User-ID).md) and Privilege-Escalation.

## Time-Based Searches

### Modified Time (`-mtime`)

- Files modified within last 1 day

```bash
find /var/log -mtime -1
```

- Files modified more than 7 days ago

```bash
find /var/log -mtime +7
```

- Files modified exactly 3 days ago

```bash
find /var/log -mtime 3
```

### Access Time (`-atime`)

- Files accessed within last 2 days

```bash
find /home -atime -2
```

### Change Time (`-ctime`)

- Files whose metadata changed within last 24 hours

```bash
find /etc -ctime -1
```

### Minute-Based Searches

- Files modified within last 30 minutes

```bash
find /tmp -mmin -30
```

- Files accessed within last 10 minutes

```bash
find /tmp -amin -10
```

> [!NOTE]
> **Reading time signs**
> With time tests, `-n` means "within the last n units", `+n` means "more than n units ago", and a bare `n` means "exactly n units ago".

## Size-Based Searches

- Files larger than 100 MB

```bash
find / -type f -size +100M
```

- Files smaller than 10 KB

```bash
find / -type f -size -10k
```

- Files exactly 1 GB

```bash
find / -type f -size 1G
```

- Empty files

```bash
find / -type f -size 0
```

## Actions with `-exec`

- Find and run a command for each match

```bash
find ./Documents/ -type f -name "*.log" -exec grep 'root' {} \;
```

- Find and delete `.txt` files with confirmation

```bash
find ./Documents/ -name *.txt -exec rm -i {} \;
```

- Find and copy a matched file

```bash
find / -name password -exec cp /etc/openldap/certs/password /tmp \;
```

- Run system commands on found files

```bash
find /etc/passwd -exec date \;
```

```bash
find /etc/passwd -exec uname -a \;
```

```bash
find /etc/passwd -exec id \;
```

- Open a shell when a path matches

```bash
find /home -exec /bin/sh \;
```

```bash
find /home -exec /bin/bash \;
```

> [!WARNING]
> **`-exec` is an execution primitive**
> `find ... -exec /bin/sh \;` is a well-known privilege-escalation technique when `find` runs with elevated privileges (for example via a misconfigured SUID binary or `sudo` entry). Audit any `find` reachable as root.

### Faster `-exec` with `+`

Using `+` is faster because it passes multiple files at once.

- Delete multiple `.tmp` files efficiently

```bash
find /tmp -name "*.tmp" -exec rm {} +
```

- Run `chmod` on many files

```bash
find . -type f -name "*.sh" -exec chmod +x {} +
```

### Delete Files Directly

- Delete `.log` files

```bash
find /var/log -name "*.log" -delete
```

- Delete empty files

```bash
find /tmp -type f -empty -delete
```

> [!WARNING]
> **`-delete` is permanent**
> `-delete` permanently removes files. Test without `-delete` first (run the query, review the matches, then add the action).

## Depth & Grouping

### Depth Limits

- Limit search depth

```bash
find / -maxdepth 1 -name pass\*
```

```bash
find / -maxdepth 3 -name etc\*
```

- Minimum depth example

```bash
find /etc -mindepth 2
```

### Grouping Expressions

- Find multiple file types

```bash
find / -type f \( -name "*txt" -o -name "*log" -o -name "*html" \)
```

- Exclude specific file matches

```bash
find /tmp/nmap_outputs -maxdepth 1 -type f ! -name "*Version-Detection*"
```

## Examples

Practical, real-world one-liners.

- Copy All YAML Files

```bash
find . -name "*.yaml" -type f -exec cp -v {} /d-data/all-nuclei-templates/ \;
```

- Find Large Files

```bash
find / -type f -size +500M 2>/dev/null
```

- Find Broken Symlinks

```bash
find / -xtype l 2>/dev/null
```

- Find Files Without Owner

```bash
find / -nouser
```

- Find Files Without Group

```bash
find / -nogroup
```

- Count `.log` files

```bash
find /var/log -name "*.log" | wc -l
```

### Common Command Combinations

- `find` + `grep`

```bash
find /etc -type f -name "*.conf" -exec grep -H "root" {} \;
```

- `find` + `tar`

```bash
find /var/log -name "*.log" | tar -czvf logs.tar.gz -T -
```

- `find` + `xargs`

```bash
find /tmp -name "*.tmp" -print0 | xargs -0 rm -f
```

## Best Practices

- Use `-maxdepth` whenever possible to speed up searches.
- Redirect errors using `2>/dev/null` to keep output readable.
- Prefer specific paths instead of `/`.
- Use `-type f` or `-type d` to reduce unnecessary matches.
- Use `-exec ... +` instead of `\;` for better performance on large result sets.
- When combining with other tools, prefer `-print0 | xargs -0` to handle filenames containing spaces or newlines safely.

## Security Considerations

- `find` is the primary tool for permission audits: SUID/SGID binaries (`-perm -04000`, `-perm -g=s`), world-writable files (`-perm -o=w`), and ownerless files (`-nouser`, `-nogroup`) all map directly to CIS Benchmark and NIST hardening controls.
- Because `-exec` can launch arbitrary programs (including a shell), a `find` invocation reachable with elevated privileges is itself a privilege-escalation vector — restrict SUID/`sudo` access to it.
- `-delete` and `-exec rm` are destructive; always review the match set first. In shared or production systems, run the read-only query and inspect it before adding any mutating action.
- Broad `find /` scans can be I/O heavy; schedule intrusive audits off-peak.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Flood of "Permission denied" | Traversing unreadable paths | Append `2>/dev/null` |
| Glob matched unexpected files | Shell expanded `*` before `find` | Quote/escape the pattern (`"*.txt"`) |
| `-exec` seems to hang | Runs one process per file | Switch `\;` to `+` for batching |
| `xargs` breaks on odd filenames | Spaces/newlines in names | Use `find -print0 \| xargs -0` |
| `-newer`/time tests off by one | Sign semantics misread | Remember `-n` = within, `+n` = older than |

## Help

```bash
man find
```

```bash
find --help
```

## Related
- [File-Finding-in-Linux](File-Finding-in-Linux.md) — overview of file-search tools
- [locate](locate.md) — faster pre-indexed alternative
- [Setuid(Set-User-ID)](../Users-Groups-and-Permissions/Setuid(Set-User-ID).md) — `find -perm` hunts SUID binaries
- Privilege-Escalation — finding SUID/writable files for privesc
- [Linux Administration & Server Hardening](../Readme.md) — course hub
