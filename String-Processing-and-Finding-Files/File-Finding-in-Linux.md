# File Finding in Linux

## Overview

Locating files and binaries quickly is a fundamental Linux administration and security skill — from tracking down a stray config file, to auditing world-writable files, to finding the on-disk path of a command. Linux offers several complementary tools, each optimized for a different question:

- **`find`** — walk the live filesystem and match by name, type, time, size, owner, or permissions, and act on the results.
- **`locate`** — near-instant name lookups backed by a prebuilt database.
- **`which` / `whereis` / `type`** — resolve where a *command* lives (binary, source, man page, alias, or function).

> [!TIP]
> **Which tool when**
> - Need up-to-the-second accuracy or rich filters? → `find`
> - Need a fast name search and freshness is acceptable? → `locate`
> - Asking "where does this command come from?" → `which` / `whereis` / `type`

## Concepts

| Tool | Data source | Speed | Best for |
|---|---|---|---|
| `find` | Live filesystem walk | Slower, thorough | Attribute-based search and actions |
| `locate` | Prebuilt `mlocate`/`plocate` DB | Very fast | Name lookups when staleness is OK |
| `which` | `PATH` scan | Instant | Full path of an executable |
| `whereis` | Standard binary/man locations | Instant | Binary + source + man page |
| `type` | Shell resolution | Instant | Alias vs function vs builtin vs binary |

## find Command

The `find` command searches the filesystem in real time and can filter on virtually any file attribute, then run actions on the matches.

### Find by File Name

- Case-sensitive

```bash
find / -name filename.txt
```

- Case-insensitive

```bash
find / -iname filename.txt
```

### Find All `.conf` Files in `/etc`

```bash
find /etc -type f -name "*.conf"
```

### Find Directories Only

```bash
find /home -type d -name "backup"
```

### Find by Modified Time

- Modified within last 3 days

```bash
find /var/log -mtime -3
```

- Modified more than 30 days ago

```bash
find /var/log -mtime +30
```

### Find by Size

- Files larger than 100MB

```bash
find / -size +100M
```

- Files smaller than 10KB

```bash
find /home -size -10k
```

### Find by Owner or Group

```bash
find /var/www -user www-data
```

```bash
find /var/www -group devs
```

### Find and Delete

```bash
find /tmp -type f -name "*.log" -delete
```

### Find and Execute Command

```bash
find /var/log -name "*.gz" -exec gunzip {} \;
```

```bash
find . -type f -exec chmod 644 {} \;
```

> [!NOTE]
> **Deep dive**
> `find` has far more filters and actions (permissions, SUID/SGID hunting, `+`-batched exec, depth limits). See [Find-Command](Find-Command.md) for the full reference.

## locate Command

The `locate` command searches a prebuilt database of filenames, making it dramatically faster than `find` for simple name lookups.

```bash
locate passwd
```

```bash
locate "*.pdf"
```

> [!WARNING]
> **Database freshness**
> `locate` reads a cached index, so newly created files may not appear until the database is refreshed. Run `updatedb` (as root) first to update the locate database.

## grep with find – Search Inside Files

Combine `find` (to select files) with `grep` (to search their contents) for content-based hunting:

```bash
find . -type f -name "*.log" -exec grep "ERROR" {} +
```

## which, whereis, type

These resolve where a **command** comes from, each answering a slightly different question.

- Full path of binary

```bash
which bash
```

- Binary + source + man page

```bash
whereis bash
```

- What kind of command (alias/function/binary)

```bash
type bash
```

## Best Practices

- Prefer a specific starting path over `/` — searching the whole filesystem is slow and noisy.
- Add `-type f` or `-type d` and, where possible, `-maxdepth` to narrow `find` results and speed up the walk.
- Redirect permission errors with `2>/dev/null` to keep output readable during broad searches.
- Use `locate` for quick day-to-day name lookups and `find` when you need current, attribute-rich results.
- Keep the `locate` database current with `updatedb` (usually run automatically via cron) so results are not stale.

## Security Considerations

- File finding underpins many security audits: hunting SUID/SGID binaries, world-writable files, and files with no owner is core to Linux hardening and CIS Benchmark checks. See [Find-Command](Find-Command.md) for the permission-based queries.
- Attackers also use `find` for privilege-escalation reconnaissance (writable paths, misconfigured permissions), so understanding these queries helps both defenders and testers.
- `locate` may reveal filenames a user cannot otherwise access; on hardened systems its database can leak the existence of sensitive paths, so consider restricting it (`plocate` honors permissions better than legacy `mlocate`).

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `find` prints many "Permission denied" lines | Traversing paths you cannot read | Append `2>/dev/null` |
| `locate` returns nothing for a new file | Database not yet refreshed | Run `sudo updatedb` |
| `which` finds nothing but the command runs | It is a shell alias or function | Use `type <cmd>` instead |
| `find /` is extremely slow | Whole-filesystem walk | Narrow the path, add `-maxdepth` and `-type` |

## Related
- [Find-Command](Find-Command.md) — full `find` reference: search the filesystem by attributes
- [locate](locate.md) — fast database-backed file search
- [whereis](whereis.md) — locate binaries, sources and man pages
- [which](which.md) — find an executable in `PATH`
- [type](type.md) — classify a command (alias, function, builtin, binary)
- [Linux Administration & Server Hardening](../Readme.md) — course hub
