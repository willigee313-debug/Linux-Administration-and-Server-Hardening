# `d` (Delete Command) in `sed`

## Overview

The `d` (delete) command in `sed` removes the current **pattern space** (the line being processed) so it never reaches the output. It is one of the most common `sed` operations and is used to strip lines that match a pattern, fall inside a line-number range, or occupy a specific position such as the first or last line. Combined with the `!` negation operator, `d` can also be flipped to mean "delete everything *except*" the addressed lines.

> [!NOTE]
> By default `d` prints to stdout and leaves the source file untouched. Add `-i` (`sed -i '/pattern/d' file`) to edit the file in place.

## Concepts

### POSIX Character Classes (Used Inside `[]`)

Character classes make patterns portable and readable — they match a category of characters without listing every code point. Inside a bracket expression they are written with an extra pair of brackets, e.g. `[[:digit:]]`.

| Class | Description |
|---|---|
| `[[:upper:]]` | Uppercase characters |
| `[[:lower:]]` | Lowercase characters |
| `[[:alpha:]]` | Alphabetic characters (`A–Z`, `a–z`) |
| `[[:digit:]]` | Digits (`0–9`) |
| `[[:alnum:]]` | Alphanumeric characters |
| `[[:space:]]` | Whitespace (`space`, `tab`, `newline`) |

### Address Forms

| Form | Meaning | Example |
|---|---|---|
| `/pattern/d` | Delete lines matching a regex | `sed '/armour/d' file` |
| `Nd` | Delete line number `N` | `sed '1d' file` |
| `N,Md` | Delete lines `N` through `M` | `sed '1,3d' file` |
| `N,M!d` | Delete everything *except* `N`–`M` | `sed '1,3!d' file` |
| `$d` | Delete the last line | `sed '$d' file` |

## Commands

### Delete Lines Matching Text

- Delete lines containing the word `armour`.

```bash
sed '/armour/d' user-list.txt
```

- Delete lines where `armour` is preceded by a tab.

```bash
sed '/\tarmour/d' user-list.txt
```

- Delete lines where `armour` is preceded by a colon.

```bash
sed '/:armour/d' user-list.txt
```

- Delete lines where `armour` is preceded by any whitespace.

```bash
sed '/[[:space:]]armour/d' user-list.txt
```

### Delete Lines Matching Digits or Letters

- Delete lines containing any digit (`0–9`).

```bash
sed '/[0-9]/d' user-list.txt
```

- Delete lines containing any digit using POSIX class.

```bash
sed '/[[:digit:]]/d' user-list.txt
```

- Delete lines containing any alphabetic character.

```bash
sed '/[a-zA-Z]/d' user-list.txt
```

- Delete lines containing any uppercase letter.

```bash
sed '/[A-Z]/d' user-list.txt
```

### Delete Lines Containing Special Characters

- Delete lines containing a forward slash (`/`).

```bash
sed '/\//d' user-list.txt
```

### Delete Lines Based on Whitespace Patterns

- Delete lines where `hr` is preceded by a tab.

```bash
sed '/\thr/d' user-list.txt
```

- Delete lines where `hr` is preceded by a space.

```bash
sed '/ hr/d' user-list.txt
```

- Delete lines where `emp` is preceded by a tab.

```bash
sed '/\temp/d' user-list.txt
```

### Delete Specific Line Numbers

- Delete line `1`.

```bash
sed '1d' user-list.txt
```

- Delete lines `1` through `3`.

```bash
sed '1,3d' user-list.txt
```

- Delete everything except lines `1` through `3`.

```bash
sed '1,3!d' user-list.txt
```

### Delete Last Line Operations

- Delete only the last line.

```bash
sed '$d' user-list.txt
```

- Delete everything except the last line.

```bash
sed '$!d' user-list.txt
```

## Best Practices

- Test the match with `-n` and `p` (`sed -n '/pattern/p' file`) before deleting, so you can see exactly which lines will disappear.
- Anchor patterns (`^`, `$`) to avoid deleting unintended lines when a substring appears elsewhere.
- Prefer POSIX classes (`[[:digit:]]`) over ad-hoc ranges (`[0-9]`) for locale-safe, self-documenting scripts.

## Security Considerations

> [!WARNING]
> `sed -i '/pattern/d'` on log or audit files can destroy forensic evidence and may violate retention policy (PCI-DSS, NIST 800-92). Never delete lines from `/var/log/*` on production systems without an authorised change and an off-host copy of the log.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Nothing deleted | Pattern did not match (case, whitespace, anchors) | Verify with `sed -n '/pattern/p' file` |
| `\t` matched literally | Some shells/`sed` builds don't expand `\t` | Use a literal tab (`Ctrl-V` `Tab`) or `[[:space:]]` |
| Too many lines removed | Unanchored substring matched broadly | Add `^`/`$` anchors or `-w` semantics |

## Related
- [sed](sed.md) — parent `sed` command reference
- [p (print) and -n](p-(print-command)-and--n-option.md) — complementary select/print command
- [c (change)](c-(change-command).md) — change a line versus deleting it
- [a (append) and i (insert)](a-(append)-and-i(prepand).md) — add lines instead of removing them
- [String-Processing](String-Processing.md) — text-processing toolkit hub
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
