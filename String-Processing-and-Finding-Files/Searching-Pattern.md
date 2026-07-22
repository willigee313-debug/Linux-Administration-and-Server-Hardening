# Searching Pattern

## Overview

`awk` is not only a field extractor — it is a full **pattern-scanning language**. Every `awk` program is a sequence of `pattern { action }` rules: for each input record, `awk` evaluates the pattern and, when it matches, runs the action. Patterns can be regular expressions (`/regex/`), field comparisons (`$1=="root"`), numeric tests (`$3>=1000`), or boolean combinations of all three. This makes `awk` a precise alternative to `grep` when you need to match against **specific fields** rather than the whole line.

> [!NOTE]
> The general form is `awk 'pattern { action }' file`. If the action is omitted, the default action is `{print $0}`; if the pattern is omitted, the action runs for every record.

## Concepts

| Pattern form | Matches when | Example |
|--------------|--------------|---------|
| `/regex/` | The record matches the regular expression | `/root/` |
| `!/regex/` | The record does **not** match | `!/root/` |
| `/^root/` | Record begins with `root` | anchored start |
| `/bash$/` | Record ends with `bash` | anchored end |
| `$n=="value"` | Field `n` exactly equals a value | `$1=="root"` |
| `$n!="value"` | Field `n` is not equal to a value | `$1!="root"` |
| `$n>=N` | Numeric comparison on field `n` | `$3>=1000` |
| `$0 ~ /regex/` | Explicit regex match operator | `$0 ~ /root/` |
| `$0 !~ /regex/` | Explicit negated regex match | `$0 !~ /root/` |

- General syntax:

```bash
awk '/pattern/ {print $0}' file
```

## Commands

### Basic Pattern Searching

- Searches for lines containing `root`.

```bash
cat /etc/passwd | awk '/root/ {print $0}'
```

- Searches for lines containing a tab followed by `root`.

```bash
cat /etc/passwd | awk '/\troot/ {print $0}'
```

- Searches for lines NOT containing `root`.

```bash
cat /etc/passwd | awk '!/root/ {print $0}'
```

- Searches for lines where `root` appears at the beginning.

```bash
cat /etc/passwd | awk '/^root/ {print $0}'
```

- Searches for lines where `bash` appears at the end.

```bash
cat /etc/passwd | awk '/bash$/ {print $0}'
```

### Searching Without `cat` (Recommended)

> [!TIP]
> `awk` reads files directly, so drop the leading `cat`. It saves a process and reads more clearly — the recommended idiom for all of the searches above.

- Search for `root`.

```bash
awk '/root/ {print $0}' /etc/passwd
```

- Exclude lines containing `root`.

```bash
awk '!/root/ {print $0}' /etc/passwd
```

- Match lines starting with `root`.

```bash
awk '/^root/ {print $0}' /etc/passwd
```

- Exclude lines starting with `root`.

```bash
awk '!/^root/ {print $0}' /etc/passwd
```

- Match lines ending with `bash`.

```bash
awk '/bash$/ {print $0}' /etc/passwd
```

- Match lines ending with `/sbin/nologin`.

```bash
awk '/\/sbin\/nologin$/ {print $0}' /etc/passwd
```

- Match lines ending with `/usr/sbin/nologin`.

```bash
awk '/\/usr\/sbin\/nologin$/ {print $0}' /etc/passwd
```

- Match lines ending with `/bin/bash`.

```bash
awk '/\/bin\/bash$/ {print $0}' /etc/passwd
```

### Using Field Separator (`-F`)

- Search for lines ending with `/bin/bash`.

```bash
awk -F: '/\/bin\/bash$/ {print $0}' /etc/passwd
```

- Print only usernames whose shell is `/bin/bash`.

```bash
awk -F: '/\/bin\/bash$/ {print $1}' /etc/passwd
```

### Exact Match on Fields

> [!IMPORTANT]
> A regex like `/root/` matches `root` **anywhere** in the record, including substrings such as `chroot`. Use an exact field comparison (`$1=="root"`) when you need a precise match on a specific column.

- Print lines where username is exactly `root`.

```bash
awk -F: '$1=="root" {print $0}' /etc/passwd
```

- Print lines where username is NOT `root`.

```bash
awk -F: '$1!="root" {print $0}' /etc/passwd
```

### AWK with `ps`

- Print processes where the user is `root`.

```bash
ps -aux | awk '$1=="root" {print $0}'
```

- Print processes where the user is NOT `root`.

```bash
ps -aux | awk '$1!="root" {print $0}'
```

- Print formatted process details for `root`.

```bash
ps -aux | awk 'BEGIN{ print "USERNAME \t PID \t CMD"} $1=="root" {print $1,"\t",$2,"\t",$11,$12,$13,$14,$15}'
```

### AWK with `ifconfig`

- Print lines containing `inet`.

```bash
ifconfig | awk '/inet /{print $0}'
```

- Print lines where the first column is `inet`.

```bash
ifconfig | awk '$1=="inet" {print $0}'
```

- Print IP addresses.

```bash
ifconfig | awk '$1=="inet" {print $2}'
```

- Print MAC addresses.

```bash
ifconfig | awk '$1=="ether" {print $2}'
```

- Print interface names where the third column is `mtu`.

```bash
ifconfig | awk '$3=="mtu" {print $1}'
```

### Combining AWK with Pipes

- Check `/var/log/messages` for lines where the seventh field is `session`, then filter those containing `root`.

```bash
cat /var/log/messages | awk '$7=="session" {print $0}' | awk '/root/{print $0}'
```

- Recommended version without unnecessary `cat`:

```bash
awk '$7=="session" {print $0}' /var/log/messages | awk '/root/{print $0}'
```

### Multiple Conditions

- Print lines where username is `root` and shell is `/bin/bash`.

```bash
awk -F: '$1=="root" && $7=="/bin/bash" {print $0}' /etc/passwd
```

- Print lines where UID is greater than or equal to `1000`.

```bash
awk -F: '$3>=1000 {print $0}' /etc/passwd
```

- Print users whose shell is not `/sbin/nologin`.

```bash
awk -F: '$7!="/usr/sbin/nologin" {print $1}' /etc/passwd
```

### Case-Insensitive Search

- Search for `root` ignoring case sensitivity.

```bash
awk 'tolower($0) ~ /root/ {print $0}' /var/log/messages
```

### Using Regular Expression Match Operator

- Search using the `~` operator.

```bash
awk '$0 ~ /root/ {print $0}' /etc/passwd
```

- Negated match using `!~`.

```bash
awk '$0 !~ /root/ {print $0}' /etc/passwd
```

## Security Considerations

> [!WARNING]
> These patterns are staples of both auditing and post-exploitation. Filtering `/etc/passwd` for `$7=="/bin/bash"` reveals which accounts have interactive shells; UID `>=1000` isolates human user accounts; scanning `/var/log/messages` for `root` sessions surfaces privileged authentication events. When hardening, ensure service accounts use `/usr/sbin/nologin` and audit any unexpected interactive shells.

## Related
- [awk-Command](awk-Command.md) — awk language overview
- [print-BEGIN{}-{}-END{}](print-BEGIN{}-{}-END{}.md) — act on matched lines
- [Field-Separator](Field-Separator.md) — match against specific fields
- [grep-Command](grep-Command.md) — line-oriented pattern matching alternative
- [String-Processing](String-Processing.md) — text-filtering toolkit
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
