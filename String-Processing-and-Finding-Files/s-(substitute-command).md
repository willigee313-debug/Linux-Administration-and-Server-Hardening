# s (substitute command)

## Overview

`s` is the **substitute** command of `sed` — by far the most heavily used sed operation. It matches a regular expression in the current line and replaces the matched text with a replacement string. Because it works line by line on a stream, it is ideal for search-and-replace across large files, log processing, and generating structured text (such as `username:password` pairs) inside shell pipelines and automation.

- Replaces text that matches a pattern with a replacement string.
- By default, only the **first match on each line** is replaced.
- Add the `g` flag for **global** (all matches per line) replacement.

> [!TIP]
> `s` is a single sed command. For the surrounding tool — addressing, options such as `-i` and `-r`, and the other commands (`p`, `d`, `a`, `i`, `c`) — see the parent note [sed](sed.md).

---

## Concepts

### Syntax

```bash
sed 's/pattern/replacement/'
```

| Component | Description |
|---|---|
| `s` | The substitute command |
| `pattern` | Regular expression to match |
| `replacement` | Text that replaces the match |
| `/` | Delimiter (any character may be used instead) |
| *flags* | Modifiers after the closing delimiter (`g`, `I`, a number, `p`, …) |

### Substitution Flags

| Flag | Effect |
|---|---|
| *(none)* | Replace only the **first** match on each line |
| `g` | Replace **all** matches on each line (global) |
| `I` | Case-**insensitive** match (GNU sed) |
| `p` | Print the line if a substitution was made (pair with `-n`) |
| `N` (a number) | Replace only the **Nth** match on the line |

### Regex Quantifiers in Basic Mode

`sed` uses **basic regular expressions** by default, so `+`, `?`, and `|` must be backslash-escaped (`\+`) to act as quantifiers. Switch to extended regex with `sed -r` (or `-E`) to drop the backslashes.

| Token | Meaning (basic regex) |
|---|---|
| `[[:digit:]]` | Any single digit (POSIX class) |
| `[0-9]` | Any single digit (range) |
| `\+` | One or more of the preceding token |
| `&` | The entire matched text (in the replacement) |
| `.*` | Any run of characters (greedy) |

---

## Examples

### Basic Substitution

- Replace `five` with `two` (first match only).

```bash
echo "one five three" | sed 's/five/two/'
```

- Replace `armour` with `ARMOUR` in `user-list.txt`.

```bash
sed 's/armour/ARMOUR/' user-list.txt
```

### Replace Digits Using Regex

- Replace a sequence of digits with `***`.

```bash
echo "Armour user UID 1000" | sed 's/[[:digit:]]\+/***/'
```

- Replace the first digit with `****`.

```bash
echo "Armour user UID 11415" | sed 's/[[:digit:]]/****/'
```

- Replace one or more digits using `[0-9]`.

```bash
echo "Armour user UID 1000" | sed 's/[0-9]\+/****/'
```

- Replace the first digit in each line of `user-list.txt`.

```bash
sed 's/[0-9]/****/' user-list.txt
```

### Replace Tabs and Numbers

- Replace a tab followed by digits with `****`.

```bash
sed 's/\t[0-9]\+/****/' user-list.txt
```

### Replace Exact Values

- Replace the literal `1000` with `****`.

```bash
echo "armour user UID 1000" | sed 's/1000/****/'
```

### Global Replacement with `g`

- Replace only the first `0` with `1` (default behaviour).

```bash
echo "0 2 5 9 0 4" | sed 's/0/1/'
```

- Replace **all** `0`s with `1` using the `g` flag.

```bash
echo "0 2 5 9 0 4" | sed 's/0/1/g'
```

### Replace Text in Files

- Replace the first occurrence of `armour` with `Armour` on each line.

```bash
sed 's/armour/Armour/' user-list.txt
```

- Replace `armour` with `Armour` on all lines using an explicit address range.

```bash
sed '1,$s/armour/Armour/' user-list.txt
```

### Replace Numeric Patterns

- Replace numbers containing two or more digits with `***`.

```bash
sed 's/[1-9][[:digit:]]/***/' /etc/passwd
```

```bash
sed 's/[1-9][[:digit:]]\+/***/' /etc/passwd
```

### Backreference with `&`

`&` in the replacement expands to the **entire matched text**, so you can wrap, repeat, or decorate the match without retyping it.

- Repeat the matched word multiple times.

```bash
sed 's/armour/Armour & & & &/' /etc/passwd
```

> Given the input:

```text
armour
```

> the line becomes:

```text
Armour armour armour armour armour
```

### Combine `s` with `q`

- Quit with different exit codes depending on whitespace conditions.

```bash
sed -e '/[[:blank:]]\+$/q9' -e '/^[[:blank:]]\+/q7' user-list.txt
```

---

## Practical Formatting Examples

### Prefix Each Line

- Add `admin:` before every line in `user-list.txt`.

```bash
sed "s/.*/admin:&/" user-list.txt
```

### Append Text to Each Line

- Add `:password123` after every line in `user-list.txt`.

```bash
sed "s/.*/&:password123/" user-list.txt
```

### Combine Users and Passwords

- Loop through usernames, prefixing each line.

```bash
for user in $(cat user.txt); do sed "s/.*/$user:&/" user-list.txt; done
```

- Loop through passwords, appending each to each line.

```bash
for password in $(cat password.txt); do sed "s/.*/&:$password/" user.txt; done
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: Terminal running echo "one five three" | sed 's/five/two/' and a global substitution with the g flag, showing first-match-only versus all-matches replacement_

---

## Best Practices

> [!TIP]
> Always dry-run a substitution and inspect the output before writing it back to disk. When you are ready to modify a file in place, use `sed -i.bak 's/old/new/g' file` so a `.bak` copy is kept for rollback.

- Choose an **alternate delimiter** when the pattern or replacement contains `/` — for example `sed 's|/old/path|/new/path|g'` — instead of escaping every slash.
- Use `-r` (extended regex) to avoid backslash-heavy quantifiers such as `\+`, `\?`, and `\|`.
- Remember `s` replaces only the **first** match per line unless you add the `g` flag.

---

## Security Considerations

- The credential-generation one-liners here (`&:password123`, `$user:&`) produce example data only. Never hard-code real passwords in scripts or shell history; source them from a secrets manager or a permission-restricted file.
- Do not interpolate untrusted input directly into a sed script — a crafted string can inject its own delimiter or additional commands. Validate and quote input first.
- Editing sensitive files such as `/etc/passwd` with `s` and `-i` can corrupt authentication if the pattern is wrong; back up first (`-i.bak`) and verify the result before relying on it.

---

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Only the first match on a line is replaced | `g` flag missing | Append `g`: `s/old/new/g` |
| `+`, `?`, `(` treated literally | Basic regex mode | Escape as `\+` `\?` `\(`, or run `sed -r` |
| `unknown option to 's'` | Unescaped `/` inside the pattern | Use an alternate delimiter, e.g. `s|old|new|` |
| `&` printed literally instead of the match | `&` was escaped as `\&` | Use a bare `&` to expand to the matched text |

---

## References

| Resource | Command |
|---|---|
| sed manual page | `man sed` |
| GNU sed full reference | `info sed` |

---

## Related

- [sed](sed.md) — parent Stream Editor note (options, addressing, and all other commands).
- [-e-option-Run-multiple-sed-commands](-e-option-Run-multiple-sed-commands.md) — chain multiple substitutions in one invocation.
- [-i-option-Changing-files-for-sure](-i-option-Changing-files-for-sure.md) — write substitutions back to the file in place.
- [grep-Command](grep-Command.md) — regular-expression line-matching companion.
- [String-Processing](String-Processing.md) — text-processing hub for this module.
- [Linux Administration & Server Hardening](../Readme.md) — course hub.
