# Number of Records (`NR`)

## Overview

`NR` is a built-in `awk` variable that holds the **Number of Records** processed so far. By default `awk` treats each line as one record, so `NR` doubles as the current **line number**. It increments automatically as `awk` reads input and continues counting **across multiple input files**.

Two idioms dominate everyday use: printing `NR` on each line to number output, and `END { print NR }` to report the **total line count** once all input is consumed.

> [!TIP]
> **`NR` vs `NF`**
> `NR` counts **records** (lines) — the vertical position. `NF` counts **fields** within the current record — the horizontal width. They are frequently combined to annotate output with both line number and field count.

## Concepts

| Expression | Meaning |
|---|---|
| `NR` | Current record (line) number, starting at `1` |
| `END { print NR }` | Total number of records after all input is read |
| `NR, $0` | Line number followed by the whole line |
| `NR` across files | Keeps counting; does not reset per file (use `FNR` for per-file) |

## Basic `NR` Examples

- Prints the **record number** (line number) for the input.

```bash
echo "one two three four" | awk '{print NR}'
```

- Prints the **line number** for each line in `/etc/passwd`.

```bash
cat /etc/passwd | awk '{print NR}'
```

- Preferred syntax without `cat`.

```bash
awk '{print NR}' /etc/passwd
```

- Prints the **total number of lines** (records) at the end of the file.

```bash
cat /etc/passwd | awk 'END {print NR}'
```

- Preferred syntax without `cat`.

```bash
awk 'END {print NR}' /etc/passwd
```

> [!NOTE]
> **A quicker line count**
> `awk 'END {print NR}' file` is a handy line counter, equivalent in most cases to `wc -l < file`. `awk` counts records even when the final line lacks a trailing newline.

## Combine `NR` with `NF`

Pairing `NR` (line number) with `NF` (field count) is a common way to annotate structured output.

- Displays line number (`NR`) along with number of fields (`NF`) in each line.

```bash
cat /etc/passwd | awk '{print "Number of fields in line " NR ": " NF}'
```

- With field separator (`:`).

```bash
cat /etc/passwd | awk -F ":" '{print "Number of fields in line " NR ": " NF}'
```

- Preferred syntax without `cat`.

```bash
awk -F ":" '{print "Number of fields in line " NR ": " NF}' /etc/passwd
```

## AWK with `ifconfig`

- Displays **line number (`NR`)** and **number of fields (`NF`)** for each line.

```bash
ifconfig | awk '{print "Number of fields " NF " in line No.: " NR}'
```

- Prints only the **line number**.

```bash
ifconfig | awk '{print NR}'
```

- Prints both **line number** and the complete line.

```bash
ifconfig | awk '{print NR, $0}'
```

## Best Practices

- `NR` starts counting from `1` and increases automatically for every input line processed — never set it manually.
- Remember that `NR` represents the current record number **across all input files** combined; use `FNR` when you need a count that resets at the start of each file.
- `END {print NR}` is the idiomatic way to count total lines in a file.
- Prefer `awk '...' file` over `cat file | awk '...'` — `awk` reads files directly.

> Example:

```bash
awk 'END {print NR}' /etc/passwd
```

## Related
- [awk-Command](awk-Command.md) — awk language overview
- [Number-of-Fields](Number-of-Fields.md) — companion `NF` variable
- [Record-Separator](Record-Separator.md) — defines what counts as a record
- [$1-$2-Dollars-everywhere]($1-$2-Dollars-everywhere.md) — fields within each record
- [Linux Administration & Server Hardening](../Readme.md) — course hub
